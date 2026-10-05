---
title: "LockBit 5.0을 격리 랩에서 돌렸더니 pcap이 비어 있었다"
date: 2026-10-04
draft: false
tags: ["LockBit", "ransomware", "sandbox", "REMnux", "FlareVM"]
description: "파일은 전부 암호화됐는데 네트워크 캡처에는 악성 통신이 없었던 이유를 공개 분석과 대조해 확인했다."
---

> 이 글의 샘플은 공유하지 않습니다. 해시와 공개 분석 링크만 적습니다.

## 한 줄 요약

격리망에서 LockBit 5.0 Windows 샘플을 실행하자 VM의 파일은 전부 암호화됐지만, 패킷 캡처에는 악성 통신이 없었다. 공개 샌드박스 보고서도 같은 결과였다.

## 샘플

- 패밀리: LockBit 5.0 (Windows EXE, 691 KB)
- SHA256: `180e93a091f8ab584a827da92c560c78f468c45f2539f73ab2deb308fb837b38`
- 처음에는 이름을 "rocbit5"로 전달받았지만 해시 조회로 LockBit 5.0임을 확인했다.

## 랩 구성

```
[FlareVM .20] --- VMnet10 (격리) --- [REMnux .10]
                                      dnsmasq: 모든 도메인 → .10
                                      INetSim: 가짜 HTTP/HTTPS
                                      tcpdump: pcap 캡처
```

- 호스트 어댑터를 끄고 DHCP 없이 고정 IP만 사용했다.
- 샘플은 읽기 전용 ISO로 반입했고 공유 폴더는 연결하지 않았다.
- 실행 전에 스냅샷을 찍고, 실행 뒤에 되돌렸다.

## 결과

| 관찰 | 내용 |
|---|---|
| 로컬 | VM의 파일 전체 암호화 |
| DNS | Microsoft 도메인 7종 (인증서 갱신, 텔레메트리) |
| HTTP/TLS | `Microsoft-CryptoAPI`, `*.microsoft.com`, `*.office.com` |
| ICMP | 1.1.1.1 등으로 분당 약 24회, 캡처 내내 일정 (배경 동작) |
| 스캔·비표준 포트·하드코딩 IP | 없음 |

## 왜 통신이 없을까

- 암호화는 VM 안에서 끝나는 로컬 작업이라 네트워크가 필요 없다.
- 협상은 몸값 메모의 Tor 주소로 피해자가 직접 한다.
- 공개 Triage 보고서에도 이 해시의 네트워크 항목이 없다.

한계도 있다. LockBit 계열은 횡적 이동에 ARP/SMB를 쓰는 것으로 알려져 있지만, 상대 장비가 없는 랩에서는 스캔이 실행되지 않았다는 자료가 있다. 이번 랩에는 상대 VM이 없어서 이 부분은 확인하지 못했다.

## 삽질 기록

| 증상 | 원인 | 해결 |
|---|---|---|
| `nslookup` 무응답 | INetSim 1.3.2의 DNS 서비스가 시작 직후 종료 | dnsmasq로 대체 |
| INetSim이 "already running" | 남은 PID 파일 | `/var/run/inetsim.pid` 삭제 |
| 암호 ZIP 복사 시 `0x80004005` | 탐색기가 AES ZIP을 처리 못 함 | 7-Zip 사용 |
| FlareVM 붙여넣기 불가 | VMware Tools 미동작 | ISO 반입 |

## 다음에 할 것

- Procmon으로 암호화 순서와 확장자 규칙 기록
- SMB 공유가 있는 상대 VM을 추가해 SMB/ARP 트래픽 캡처
- 몸값 메모 분석

## IOC

- SHA256: 위 참고
- Tor (공개 Triage 설정에서 추출, 직접 접속하지 않음):
  - `lockbitapt67g6rwzjbcxnww5efpg4qok6vpfeth7wx3okj52ks4wtad[.]onion`
  - `lockbitsuppyx2jegaoyiw44ica5vdho63m5ijjlmfb7omq3tfr3qhyd[.]onion`
  - `lockbitfbinpwhbyomxkiqtwhwiyetrbkb4hnqmshaonqxmsrqwg7yad[.]onion`
