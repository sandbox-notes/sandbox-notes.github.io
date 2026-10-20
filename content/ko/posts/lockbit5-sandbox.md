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

## 두 번째 실행: 감염 후 화면과 pcap

같은 랩에서 다른 빌드의 LockBit 5.0 샘플(SHA256 `7ea5afbc166c4e23498aa9747be81ceaf8dad90b8daa07a6e4644dc7c2277b82`)을 한 번 더 실행했다. 실행 옵션은 기록하지 못했다(옵션 목록은 아래 도움말 절 참고).

| 관찰 | 내용 |
|---|---|
| pcap | 1152 패킷, 약 12분 (19:28:51 ~ 19:41:01). 샘플 실행(약 19:29)을 포함 |
| DNS·TLS | Microsoft 도메인과 Chrome 업데이트 확인(`update.googleapis.com`)뿐 |
| 외부 연결 시도 | 없음 (핑은 이전과 같은 배경 ICMP) |
| SMB(445) | 없음 |
| 로컬 파일 | 이름 뒤에 **파일마다 다른 16자리 16진수 확장자**가 붙음 |
| 몸값 메모 | `ReadMeForDecrypt.txt` |
| 실행 파일 | 실행 후 탐색기에서 `0 KB`로 표시됨 |
| 드라이브 | 실행 직후 새 드라이브 문자(`M:`, `N:`)가 보였고, `N:`(EFI 파티션 포함)에도 몸값 메모가 생김 |

![암호화된 Downloads 폴더](/images/lockbit5-downloads-encrypted.png)

![N: 드라이브에 생성된 몸값 메모](/images/lockbit5-n-drive-note.png)

- 샘플이 **끝까지 실행되어 파일을 암호화했는데도** 캡처에는 C2나 외부 연결 시도가 하나도 없었다. 앞의 결론이 이번에도 같다.
- 확장자가 파일마다 달라서 "알려진 확장자 목록"에 의존하는 탐지는 어렵다.
- `M:`, `N:`이 샘플 때문에 생긴 것인지는 이 관찰만으로 확정할 수 없다.

## 도움말(`-h`) 출력으로 본 옵션

이 빌드를 `-h`로 실행하면 배너에 **LOCKBIT5.0 / ChuongDoung Locker v1.01 / Windows x64**가 표시되고 옵션 목록이 나온다. 위 실행에서 실제로 쓴 옵션은 기록하지 못했고, 아래는 도움말에 적힌 내용이다.

![-h 도움말 출력](/images/lockbit5-help.png)

| 구분 | 옵션 | 설명 (도움말 원문 요약) |
|---|---|---|
| 대상 | `-p <dirs>` / `-b <dirs>` | 암호화할 폴더 / 제외할 폴더 (세미콜론 구분) |
| 동작 모드 | `-i` | 확장자·메모·수정 날짜를 바꾸지 않는 invisible 모드 |
| | `-v` / `-d` | 콘솔 상태 표시 모드 / 디버그 출력 모드 |
| 메모 | `-n <0/1/2>` | 메모 저장 위치 (0 없음, 1 모든 곳, 2 `C:\`만). `-i`와 함께 쓰면 무시 |
| 암호화 | `-m <all/local/net>` | 로컬만 또는 네트워크만 암호화 |
| | `-f` / `-w` | 빠른 암호화 / 암호화 후 빈 공간 덮어쓰기 |
| 기타 | `-k` | `.exe`를 삭제하지 않음 |
| | `-nomutex` | 여러 인스턴스 허용 |
| | `-t <초>` | 암호화 시작 전 대기 시간 |

- 명령줄 길이는 500자로 제한된다.
- 위 두 번째 실행에서 **실행 파일이 `0 KB`로 보인 것**은 `-k`(.exe를 삭제하지 않음) 옵션이 존재한다는 점과 맞물려 보이지만, 어떤 옵션으로 실행했는지 기록이 없어서 **원인은 확정하지 못했다.**
- `-m net`이 있어서 네트워크 공유가 있는 환경에서는 SMB 트래픽이 달라질 수 있다. 이 랩에는 상대 VM이 없어 확인하지 못했다.
- 스크린샷 속 문자열은 샘플이 출력한 도움말 그대로다.

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
