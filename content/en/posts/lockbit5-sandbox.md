---
title: "I Ran LockBit 5.0 in an Isolated Lab and the PCAP Was Empty"
date: 2026-10-04
draft: false
tags: ["LockBit", "ransomware", "sandbox", "REMnux", "FlareVM"]
description: "Every file was encrypted, yet the packet capture showed no malicious traffic. Here is why, checked against public analyses."
---

> Translated from the Korean original. The sample itself is never shared; only the hash and links to public analyses are.

## TL;DR

I detonated a LockBit 5.0 Windows sample on an isolated network. All files in the VM were encrypted, but the packet capture contained no malicious traffic. Public sandbox reports for the same hash agree.

## Sample

- Family: LockBit 5.0 (Windows EXE, 691 KB)
- SHA256: `180e93a091f8ab584a827da92c560c78f468c45f2539f73ab2deb308fb837b38`
- It was first handed to me as "rocbit5"; a hash lookup showed it is LockBit 5.0.

## Lab

```
[FlareVM .20] --- VMnet10 (isolated) --- [REMnux .10]
                                          dnsmasq: every domain -> .10
                                          INetSim: fake HTTP/HTTPS
                                          tcpdump: packet capture
```

- Host adapter disabled, no DHCP, static IPs only.
- The sample was delivered as a read-only ISO; no shared folder was attached to the detonation VM.
- A snapshot was taken before the run and restored afterwards.

## Results

| Observation | Detail |
|---|---|
| Local | All files in the VM encrypted |
| DNS | 7 Microsoft domains (certificate refresh, telemetry) |
| HTTP/TLS | `Microsoft-CryptoAPI`, `*.microsoft.com`, `*.office.com` |
| ICMP | About 24 pings/min to 1.1.1.1 and friends, constant for the whole capture (background) |
| Scans, odd ports, hard-coded IPs | None |

## Why no traffic?

- Encryption is a local operation and needs no network.
- Negotiation happens when the victim visits the Tor address in the ransom note.
- Public Triage reports for this hash list no network activity either.

One caveat: LockBit variants are known to use ARP/SMB for lateral movement, but reports say the scan does not run in a lab with no reachable peers. My lab had no peer VM, so I could not test that.

## Second run: after infection, and the pcap

I ran a different build of LockBit 5.0 (SHA256 `7ea5afbc166c4e23498aa9747be81ceaf8dad90b8daa07a6e4644dc7c2277b82`) in the same lab. I did not record the command-line options used.

| Observation | Detail |
|---|---|
| pcap | 1,152 packets, about 12 minutes (19:28:51 to 19:41:01), covering the execution (around 19:29) |
| DNS / TLS | Only Microsoft domains and a Chrome update check (`update.googleapis.com`) |
| Outbound connection attempts | None (pings are the same background ICMP as before) |
| SMB (445) | None |
| Local files | A **different random 16-hex-digit extension per file** was appended |
| Ransom note | `ReadMeForDecrypt.txt` |
| Executable | Showed as `0 KB` in Explorer after the run |
| Drives | New drive letters (`M:`, `N:`) appeared right after execution, and a note was also dropped on `N:` (which holds an EFI partition) |

![Encrypted Downloads folder](/images/lockbit5-downloads-encrypted.png)

![Ransom note on the N: drive](/images/lockbit5-n-drive-note.png)

- The sample **ran to completion and encrypted files**, yet the capture shows no C2 and no outbound connection attempts, matching the first run.
- Because the extension differs per file, detection that relies on a list of known extensions would struggle.
- These observations alone do not prove that the sample created `M:` and `N:`.

## Lessons learned

| Symptom | Cause | Fix |
|---|---|---|
| `nslookup` got no answer | INetSim 1.3.2's DNS service died right after starting | Replaced it with dnsmasq |
| INetSim said "already running" | Stale PID file | Delete `/var/run/inetsim.pid` |
| `0x80004005` copying an encrypted ZIP | Explorer cannot handle AES ZIPs | Use 7-Zip |
| Could not paste into FlareVM | VMware Tools not running | Deliver files via ISO |

## Next

- Record encryption order and extension rules with Procmon
- Add a peer VM with an SMB share and capture SMB/ARP traffic
- Analyze the ransom note

## IOCs

- SHA256: see above
- Tor (extracted from a public Triage config; I did not connect to them):
  - `lockbitapt67g6rwzjbcxnww5efpg4qok6vpfeth7wx3okj52ks4wtad[.]onion`
  - `lockbitsuppyx2jegaoyiw44ica5vdho63m5ijjlmfb7omq3tfr3qhyd[.]onion`
  - `lockbitfbinpwhbyomxkiqtwhwiyetrbkb4hnqmshaonqxmsrqwg7yad[.]onion`
