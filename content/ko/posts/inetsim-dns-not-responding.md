---
title: "INetSim DNS가 응답하지 않을 때: 'started'인데 53번 포트가 비어 있었다"
date: 2026-10-05
draft: false
tags: ["INetSim", "dnsmasq", "REMnux", "troubleshooting", "malware-analysis", "DNS"]
description: "격리 랩에서 nslookup이 계속 실패했다. 원인은 INetSim의 DNS 프로세스가 시작 직후 죽는 것과, FlareVM의 DNS 설정이 자기 자신을 가리킨 것, 두 가지였다. 진단 순서와 대체 방법을 정리했다."
---

> 이 글의 모든 명령과 출력은 제가 격리 랩(REMnux가 가짜 인터넷, FlareVM이 샘플 실행)에서 직접 겪은 것입니다. 원인 중 확인하지 못한 부분은 "추정"이라고 표시했습니다.

## 증상

FlareVM에서 `nslookup example.com`을 하면 이렇게 나왔습니다.

```
Server:  UnKnown
Address:  192.168.100.10

*** UnKnown can't find example.com: No response from server
```

- REMnux(`192.168.100.10`)로 `ping`은 **됩니다.** 네트워크는 정상이라는 뜻입니다.
- REMnux에서 INetSim은 `Simulation running.`이라고 표시합니다.
- 그런데 DNS만 응답이 없습니다.

## 진단 순서 (계층별로 좁히기)

"ping은 되는데 DNS만 안 된다"면 네트워크가 아니라 **DNS 서비스나 클라이언트 설정** 문제입니다. 아래 순서로 좁혔습니다.

### 1. REMnux: 53번 포트를 누가 열고 있나

```bash
sudo ss -ulnp | grep -E ':53\b'
```

**아무것도 나오지 않았습니다.** INetSim이 DNS를 열지 않았다는 뜻입니다.

### 2. 설정은 맞는가

```bash
grep -nE '^(start_service dns|service_bind_address|dns_default_ip)' /etc/inetsim/inetsim.conf
```

- `start_service dns`는 켜져 있음
- `service_bind_address 192.168.100.10`, `dns_default_ip 192.168.100.10`도 맞음

설정 문제는 아니었습니다.

### 3. INetSim 로그는 뭐라고 하나

```bash
sudo grep -i dns /var/log/inetsim/main.log | tail
```

```
* dns_53_tcp_udp started (PID 3347)
* smtp_25_tcp failed!
```

**`dns`는 `started`라고 찍혀 있습니다.** 그런데 포트는 비어 있습니다. 어디서 어긋난 걸까요.

### 4. 프로세스는 살아 있나

```bash
pgrep -af inetsim
```

`inetsim_http_80_tcp`, `inetsim_https_443_tcp`, `inetsim_ftp_21_tcp` 같은 서비스 프로세스는 모두 보였지만, **`inetsim_dns_53_…`만 없었습니다.** 로그에는 시작됐다고 했는데 프로세스가 없다면, **시작한 직후 죽은 것**입니다.

`started (PID …)`는 프로세스를 만든 순간에 찍는 로그라서, 그 뒤에 살아 있는지는 보장하지 않습니다.

### 5. 시작할 때 화면을 자세히 본다

```bash
sudo inetsim
```

시작 출력에 이 경고가 있었습니다.

```
deprecated method; prefer start_server() at /usr/share/perl5/INetSim/DNS.pm line 69.
Attempt to start Net::DNS::Nameserver in a subprocess at ... DNS.pm line 69.
* dns_53_tcp_udp - started (PID 4028)
```

INetSim 1.3.2(2020-05-19)의 DNS 모듈이 호출하는 `Net::DNS::Nameserver`의 방식이 **deprecated**라는 경고입니다. 새 `Net::DNS`와 이 INetSim 버전의 DNS 모듈이 맞지 않아서 프로세스가 죽는 것으로 **추정**합니다. (`Net::DNS` 버전은 확인하지 못했습니다. 이 랩은 예전에는 같은 구성으로 잘 동작했는데, 중간에 무엇이 바뀌었는지는 모릅니다.)

## 해결: DNS는 dnsmasq가 맡게 한다

INetSim은 HTTP, HTTPS 같은 나머지 서비스만 맡고, **DNS는 `dnsmasq`로 대체**했습니다.

```bash
sudo systemctl stop systemd-resolved
sudo dnsmasq --no-daemon --address=/#/192.168.100.10 \
  --listen-address=192.168.100.10 --bind-interfaces
```

- `--address=/#/192.168.100.10`: **모든 도메인을** `192.168.100.10`으로 풀어 줍니다.
- 포트 충돌을 피하려고 INetSim 설정에서 `start_service dns`를 주석 처리합니다.
- 켜 둔 터미널은 닫지 않습니다(`--no-daemon`이라 닫으면 꺼집니다).

확인:

```
nslookup example.com
Name:    example.com
Address:  192.168.100.10
```

## 그런데 또 안 됐다: 두 번째 원인

dnsmasq를 켠 뒤에도 FlareVM에서 `nslookup`이 응답하지 않는 날이 있었습니다. REMnux에서는 이렇게 확인됐습니다.

```
sudo ss -ulnp | grep ':53 '
UNCONN 0 0 192.168.100.10:53 0.0.0.0:* users:(("dnsmasq",pid=10071,fd=4))
```

dnsmasq는 정상으로 떠 있었습니다. 그럼 클라이언트 쪽입니다. FlareVM에서 `ipconfig /all`을 보니:

```
Default Gateway . . . : 192.168.100.10
DNS Servers . . . . . : 192.168.100.20
```

**DNS 서버가 FlareVM 자기 자신(`.20`)으로 설정되어 있었습니다.** 스냅샷을 되돌린 뒤 설정이 바뀐 것으로 보입니다. 관리자 권한 CMD에서 고쳤습니다.

```
netsh interface ip set dns "Ethernet0" static 192.168.100.10
ipconfig /flushdns
```

처음에는 **값이 바뀌지 않은 채로** 다시 확인했습니다. `ipconfig /all`에서 `DNS Servers`가 실제로 `.10`으로 바뀐 것을 확인하기 전에는 성공했다고 보면 안 됩니다. 명령은 **관리자 권한** 창에서 실행해야 합니다.

> 클라이언트 설정을 우회해서 서버만 확인하는 방법도 있습니다.
> ```
> nslookup example.com 192.168.100.10
> ```
> 이 명령은 어떤 DNS가 설정되어 있든 REMnux에 직접 묻습니다. 서버가 응답하는지 클라이언트 설정이 문제인지 가를 수 있습니다.

## 같이 만난 문제들

| 증상 | 원인 | 해결 |
|---|---|---|
| `PIDfile ... exists - INetSim already running?` | 비정상 종료 뒤 남은 PID 파일 | `sudo pkill -f inetsim` 후 `sudo rm -f /var/run/inetsim.pid` |
| 서비스 대부분이 `failed! Address already in use` | INetSim이 이미 떠 있는데 하나 더 실행 (TCP 포트가 이미 사용 중, UDP는 `started`로 나옴) | 모두 종료하고 **하나만** 실행 |
| `smtp_25_tcp failed!` | REMnux의 postfix가 25번 포트 사용 | `sudo systemctl stop postfix` |
| `Warning: Unknown option 'https_ssl_dn'` | 설정 파일에 이 버전이 모르는 옵션 | 동작에는 영향이 없었음 |

그리고 하나 더, **진단하다가 낸 실수**입니다.

```bash
sudo inetsim 2>&1 | head -60
```

이렇게 출력을 `head`로 자르면 60줄을 채운 뒤 파이프가 닫혀서 **INetSim이 같이 종료될 수 있습니다.** 시작 로그를 보려면 파일로 받으세요.

```bash
sudo inetsim > ~/inetsim-run.log 2>&1 &
```

## 정리: 이 순서로 보세요

1. `ping`이 되는가 → 되면 네트워크는 정상
2. `sudo ss -ulnp | grep :53` → 53번 포트를 누가 쓰나
3. `pgrep -af inetsim` → DNS 프로세스가 **살아 있나** (`started` 로그는 증거가 아님)
4. `nslookup example.com 192.168.100.10` → 서버가 응답하나 (클라이언트 설정 우회)
5. `ipconfig /all` → 클라이언트의 DNS 서버 값
6. `sudo tcpdump -ni ens33 port 53` → 요청이 도착하는지, 응답이 나가는지

## 한계

- 원인 추정(새 `Net::DNS`와 INetSim 1.3.2의 DNS 모듈 불일치)은 **확인하지 않았습니다.** 버전 비교나 패치로 확인하지 않고 dnsmasq로 대체했습니다.
- 이 랩은 REMnux와 FlareVM 두 대만 있는 단순한 구성입니다.

## 같이 보면 좋은 글

- [VMware로 악성코드 격리 랩 만들기](/ko/posts/isolated-malware-lab-vmware/)
- [LockBit 5.0을 격리 랩에서 돌렸더니 pcap이 비어 있었다](/ko/posts/lockbit5-sandbox/)
