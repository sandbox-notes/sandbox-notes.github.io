---
title: "When INetSim's DNS Does Not Answer: 'started' but Port 53 Is Empty"
date: 2026-10-05
draft: false
tags: ["INetSim", "dnsmasq", "REMnux", "troubleshooting", "malware-analysis", "DNS"]
description: "In my isolated lab, nslookup kept failing. There were two causes: INetSim's DNS process dying right after start, and the Windows VM's DNS pointing at itself. Here is the diagnosis order and the workaround."
---

> Every command and output here is something I hit myself in an isolated lab (REMnux as the fake internet, FlareVM running samples). Where I could not confirm the cause, I mark it as a guess.

## Symptom

On FlareVM, `nslookup example.com` printed:

```
Server:  UnKnown
Address:  192.168.100.10

*** UnKnown can't find example.com: No response from server
```

- `ping` to REMnux (`192.168.100.10`) **works**, so the network is fine.
- INetSim on REMnux says `Simulation running.`
- Yet DNS alone gets no answer.

## Diagnosis order (narrow it down layer by layer)

If "ping works but DNS does not", the problem is the **DNS service or the client settings**, not the network. This is how I narrowed it down.

### 1. On REMnux: who has port 53 open?

```bash
sudo ss -ulnp | grep -E ':53\b'
```

**Nothing came back.** INetSim was not listening for DNS.

### 2. Is the configuration right?

```bash
grep -nE '^(start_service dns|service_bind_address|dns_default_ip)' /etc/inetsim/inetsim.conf
```

- `start_service dns` was enabled
- `service_bind_address 192.168.100.10` and `dns_default_ip 192.168.100.10` were correct

So it was not a configuration problem.

### 3. What does INetSim's log say?

```bash
sudo grep -i dns /var/log/inetsim/main.log | tail
```

```
* dns_53_tcp_udp started (PID 3347)
* smtp_25_tcp failed!
```

**DNS says `started`.** But the port is empty. Where does it go wrong?

### 4. Is the process alive?

```bash
pgrep -af inetsim
```

The processes for `inetsim_http_80_tcp`, `inetsim_https_443_tcp`, `inetsim_ftp_21_tcp` and the rest were there, but **`inetsim_dns_53_…` was missing.** If the log says it started and there is no process, it **died right after starting**.

`started (PID …)` is logged at the moment the process is created, so it says nothing about whether it stayed alive.

### 5. Read the startup output closely

```bash
sudo inetsim
```

The startup output contained this warning:

```
deprecated method; prefer start_server() at /usr/share/perl5/INetSim/DNS.pm line 69.
Attempt to start Net::DNS::Nameserver in a subprocess at ... DNS.pm line 69.
* dns_53_tcp_udp - started (PID 4028)
```

It says the way INetSim 1.3.2 (2020-05-19) calls `Net::DNS::Nameserver` is **deprecated**. My **guess** is that a newer `Net::DNS` and this INetSim version's DNS module do not get along, which makes the process die. (I did not check the `Net::DNS` version. The same setup used to work in this lab and I do not know what changed in between.)

## Fix: let dnsmasq handle DNS

INetSim keeps the other services (HTTP, HTTPS and so on) and **dnsmasq replaces its DNS**.

```bash
sudo systemctl stop systemd-resolved
sudo dnsmasq --no-daemon --address=/#/192.168.100.10 \
  --listen-address=192.168.100.10 --bind-interfaces
```

- `--address=/#/192.168.100.10`: resolves **every domain** to `192.168.100.10`.
- Comment out `start_service dns` in INetSim's config to avoid a port clash.
- Keep that terminal open (`--no-daemon` means closing it stops dnsmasq).

Check:

```
nslookup example.com
Name:    example.com
Address:  192.168.100.10
```

## And it failed again: the second cause

Even after starting dnsmasq, there was a day `nslookup` on FlareVM still got no answer. On REMnux this showed:

```
sudo ss -ulnp | grep ':53 '
UNCONN 0 0 192.168.100.10:53 0.0.0.0:* users:(("dnsmasq",pid=10071,fd=4))
```

dnsmasq was running fine, so the client was next. On FlareVM, `ipconfig /all` showed:

```
Default Gateway . . . : 192.168.100.10
DNS Servers . . . . . : 192.168.100.20
```

**The DNS server was set to FlareVM itself (`.20`).** It seems the setting changed after restoring a snapshot. I fixed it from an elevated command prompt:

```
netsh interface ip set dns "Ethernet0" static 192.168.100.10
ipconfig /flushdns
```

At first the value had **not actually changed** when I checked again. Do not assume success until `ipconfig /all` shows `DNS Servers` as `.10`. Run the command in an **elevated** window.

> You can also bypass the client settings and test the server alone:
> ```
> nslookup example.com 192.168.100.10
> ```
> This asks REMnux directly no matter what DNS is configured, so it separates "the server does not answer" from "the client settings are wrong".

## Other problems I met on the way

| Symptom | Cause | Fix |
|---|---|---|
| `PIDfile ... exists - INetSim already running?` | Stale PID file after an unclean exit | `sudo pkill -f inetsim`, then `sudo rm -f /var/run/inetsim.pid` |
| Most services show `failed! Address already in use` | A second INetSim started while one was running (TCP ports were taken; UDP still said `started`) | Stop everything and run **just one** |
| `smtp_25_tcp failed!` | REMnux's postfix uses port 25 | `sudo systemctl stop postfix` |
| `Warning: Unknown option 'https_ssl_dn'` | An option this version does not know in the config | Had no effect on behavior |

And one more, **a mistake I made while debugging**:

```bash
sudo inetsim 2>&1 | head -60
```

Cutting the output with `head` closes the pipe after 60 lines and **can take INetSim down with it.** To read the startup log, write it to a file:

```bash
sudo inetsim > ~/inetsim-run.log 2>&1 &
```

## Summary: check in this order

1. Does `ping` work? → if yes, the network is fine
2. `sudo ss -ulnp | grep :53` → who owns port 53?
3. `pgrep -af inetsim` → is the DNS process **alive**? (a `started` log is not proof)
4. `nslookup example.com 192.168.100.10` → does the server answer? (bypasses client settings)
5. `ipconfig /all` → the client's DNS server value
6. `sudo tcpdump -ni ens33 port 53` → do requests arrive and do replies leave?

## Limitations

- The guessed cause (a newer `Net::DNS` not matching INetSim 1.3.2's DNS module) is **unconfirmed**. I replaced the DNS with dnsmasq instead of comparing versions or patching.
- This lab is a simple two-VM setup, REMnux and FlareVM only.

## Related posts

- [Building an Isolated Malware Lab on VMware](/en/posts/isolated-malware-lab-vmware/)
- [I Ran LockBit 5.0 in an Isolated Lab and the PCAP Was Empty](/en/posts/lockbit5-sandbox/)
