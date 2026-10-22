---
title: "Wi-Fi 8 (802.11bn UHR) — 이제는 속도가 아니라 신뢰성이다"
date: 2026-11-11T09:00:00+09:00
draft: false
series: ["Wi-Fi 신규 스펙"]
tags: ["Wi-Fi 8", "802.11bn", "UHR", "MAPC", "EasyMesh", "AP"]
description: "최고 속도를 올리지 않는 첫 Wi-Fi 세대, 802.11bn(UHR)의 목표와 핵심 기술, 그리고 AP 개발자가 지금 준비할 것."
---

> 이 글은 공개 자료를 기준으로 합니다. 802.11bn은 아직 드래프트 단계라 세부 동작과 일정은 바뀔 수 있고, 일정은 자료마다 날짜가 조금씩 다릅니다. 2026년 10월 기준입니다.

## 들어가며

Wi-Fi 8(IEEE 802.11bn)은 Wi-Fi 역사상 처음으로 최고 속도를 올리지 않는 세대다. 이름부터 UHR(Ultra High Reliability), 즉 "초고신뢰성"이다.

Wi-Fi 4에서 7까지는 대역폭을 넓히고, 변조 차수를 올리고, 스트림을 늘리는 방식으로 속도를 키워왔다. 그런데 Wi-Fi 7에서 320 MHz, 4096-QAM, 16 스트림까지 오면서 "이상적인 환경의 최고 속도"는 더 올릴 여지도, 체감할 일도 줄었다. 실제 사용자가 겪는 문제는 커버리지 끝에서 속도가 뚝 떨어지는 것, 옆집 AP 간섭, 로밍할 때 끊기는 것, 게임·화상회의의 지연 튐이다.

Wi-Fi 8은 바로 이 "나쁜 조건에서의 성능"을 정면으로 다룬다. 그리고 그 핵심 해법 중 하나가 AP끼리의 협력, 즉 Multi-AP Coordination이라는 점에서 메시·멀티 AP 제품을 만드는 입장에서는 가장 중요한 세대가 될 가능성이 크다.

## 목표: 숫자로 본 UHR

802.11bn PAR은 최고 속도 대신 "Wi-Fi 7 대비 25%"라는 체감 지표를 목표로 잡았다. 기본 골격(2.4/5/6 GHz, 최대 320 MHz, 4096-QAM)은 Wi-Fi 7과 같다.

| PAR 목표 | 의미 | 대응 기술 |
|---|---|---|
| 나쁜 신호 조건에서 처리량 25% 이상 향상 | 커버리지 끝, 장애물 많은 환경 | ELR, DRU |
| 지연 분포 95퍼센타일 25% 감소 | 평균이 아니라 "튀는 지연"을 줄임 | P-EDCA, LLI, NPCA, DSO |
| 로밍 시 패킷 손실 25% 감소 | AP 간 이동 중 끊김 | SMD (Seamless Mobility Domain) |
| 전력 소모 감소 | 웨어러블, 모바일 AP | DPS, MLPM |
| P2P 동작 개선 | 캐스팅, XR, 로컬 게임 | TXSPG, Co-CR |

눈여겨볼 건 지연 목표가 평균이 아니라 **95퍼센타일**이라는 점이다. 사용자가 짜증 내는 건 평균 10 ms가 아니라 가끔 튀는 200 ms이고, Wi-Fi 8은 그 꼬리를 자르는 걸 목표로 한다.

## 표준화 일정

2026년 10월 현재 802.11bn은 Draft 2.0 투표 단계로 알려져 있다. 자료에 따라 투표 시기가 7월 전후 또는 8월 초~9월 초로 다르게 적혀 있다. IEEE 최종 승인은 2028년 상반기, Wi-Fi Alliance 인증은 2027년 말~2028년 1월이 목표다.

| 시점 | 내용 |
|---|---|
| 2023-11 | TGbn 표준화 작업 시작 |
| 2025-07 | Draft 1.0 완성 (기술 범위 확정). 첫 투표에서 7,000건이 넘는 코멘트가 나왔다는 자료가 있음 |
| 2026 중반 | Draft 2.0 투표 (자료마다 7~9월로 다름) |
| 2027 중반 | Wi-Fi Alliance 인증 테스트 계획 확정 예정 |
| 2027-12 ~ 2028-01 | Wi-Fi Alliance Wi-Fi 8 인증 시작 목표 |
| 2028 상반기 | IEEE 802.11bn 최종 승인 목표 (자료별로 3~5월, Wikipedia는 9월로 예상) |

시장은 이미 앞서가고 있다. TP-Link는 2026년 10월 출시를 예고한 Archer 8 플랫폼을 IFA 2026에서 Archer 8 Ultra BN19000(19 Gbps급, 제조사 수치)으로 선보였고, 메시 제품인 Deco 8은 2027년 상반기 출시가 거론된다(자료마다 1분기 또는 2분기). 다만 미국 판매는 규제 문제로 불확실하다는 보도가 있다. 이건 Draft 기반 제품이라, Wi-Fi 6/7 때처럼 최종 표준·인증과의 차이는 펌웨어 업데이트로 메워야 한다.

## PHY 신기술: 커버리지 끝을 살린다

PHY 쪽 변화는 "멀리 있는 단말, 업링크가 약한 단말"을 살리는 데 집중돼 있다. 대표가 ELR과 DRU다.

### ELR (Enhanced Long Range)

저속이라도 끊기지 않게 하는 장거리 전용 PPDU 포맷이다. IoT 센서나 송신 전력이 약한 단말을 겨냥했다.

- 레거시 프리앰블 뒤에 ELR 모드를 알리는 마크 심볼 2개를 붙이고, UHR-STF/LTF를 3 dB 부스팅해서 낮은 SNR에서도 프레임을 잡는다.
- 20 MHz, 1 스트림, BPSK/QPSK(MCS0/1)만 쓰며 PHY 속도는 약 1.67 / 3.33 Mbps다.
- 20 MHz 안에서 52-tone RU를 네 번 반복해 약 6 dB 이득을 얻는다.

### DRU (Distributed Resource Unit)

업링크 링크 버짓 문제를 규제 범위 안에서 푸는 아이디어다. 6 GHz LPI처럼 PSD(MHz당 전력) 제한이 걸린 환경에서는 작은 RU로 보내면 총 송신 전력이 낮게 묶인다.

DRU는 작은 RU의 톤을 연속으로 모으지 않고 40/80 MHz 전체에 흩뿌린다. MHz당 전력 한도는 그대로 지키면서 총 전력은 키울 수 있어서, 자료에 따르면 최대 약 11 dB 전력 이득이 나온다. 업링크 OFDMA에서 여러 단말이 서로 다른 DRU를 동시에 쓸 수 있다.

### 정리

ELR과 DRU 모두 "최고 속도" 지표에는 하나도 기여하지 않는다. 대신 AP 하나 더 사서 달아야 했던 구석방을 살리는 기술이다. 메시 AP 판매량 관점에서는 양날의 검일 수도 있다.

## MAC 신기술: 지연 꼬리, 넓은 채널 낭비, 로밍 끊김

Wi-Fi 8의 실질적인 변화는 대부분 MAC에 있다. 핵심은 세 가지 문제를 푸는 것이다. 우선순위 트래픽이 가끔 오래 기다리는 문제, primary 20 MHz 하나 때문에 넓은 채널 전체가 막히는 문제, 로밍 때 끊기는 문제.

### 저지연: P-EDCA와 LLI

- **P-EDCA:** 기존 EDCA 위에 "급행 차선"을 하나 더 만든다. P-EDCA 단말은 이전 TXOP가 끝나면 AIFS를 다 기다리지 않고 DS-CTS라는 CTS 프레임으로 매체를 먼저 잡은 뒤 자기들끼리 경쟁한다. 평균 처리량이 아니라 지연 분포의 꼬리를 줄이는 게 목적이고, AP가 파라미터로 남용을 통제한다.
- **LLI (Low Latency Indication):** 다른 단말의 TXOP 안에서 응답하는 단말이 "나 급한 데이터 쌓여 있어"를 응답 프레임 비트 하나로 알린다. AP는 이걸 보고 업링크 슬롯을 빨리 준다.

### 넓은 채널 제대로 쓰기: NPCA와 DSO

- **NPCA (Non-Primary Channel Access):** 옆 BSS 때문에 primary 20 MHz가 바쁘면, 지금까지는 160/320 MHz 전체가 놀았다. NPCA는 미리 정한 다른 20 MHz를 임시 primary로 삼아 전송할 수 있게 한다. 아파트처럼 OBSS가 빽빽한 환경에서 효과가 크다.
- **DSO (Dynamic Sub-band Operation):** AP는 320 MHz인데 단말은 80 MHz만 지원하면, 지금까지는 모든 단말이 같은 80 MHz에 몰렸다. DSO는 AP가 단말을 320 MHz 안의 다른 80 MHz 조각으로 옮겨서 병렬로 서비스하게 한다. 전환은 ICF/ICR 프레임 교환으로 한다.

### 끊김 없는 로밍: SMD

SMD(Seamless Mobility Domain)는 여러 AP를 하나의 논리적 도메인으로 묶고, 관리 엔티티(SMD-ME)가 연결·인증 상태를 AP 대신 들고 있는 구조다. 로밍 시 MAC 상태, QoS 파라미터, 보안 컨텍스트를 타깃 AP로 넘기고, MLO를 이용해 새 링크를 먼저 붙인 뒤 옛 링크를 끊는다(make-before-break).

1. **Prepare:** 현재 AP가 SMD-ME를 통해 컨텍스트를 타깃 AP로 넘기고, 타깃 AP와 링크를 절전 상태로 미리 만들어 둔다.
2. **Execute:** 단말이 재인증 없이 새 링크를 활성화한다.
3. **Complete:** 연결이 안정되면 이전 AP가 임시 상태를 정리한다.

802.11r(FT)이 "재연결 시간을 줄이는" 접근이었다면, SMD는 "재연결 자체를 없애는" 접근이다. 보안 관점에서는 보안 컨텍스트가 AP 간에 이동한다는 점이 새로운 공격면이 된다.

### 절전: DPS와 MLPM

DPS(Dynamic Power Save)는 단말이나 모바일 AP가 평소엔 낮은 능력(좁은 대역폭, 적은 스트림, 낮은 MCS)으로 대기하다가, ICF를 받으면 고성능 모드로 전환하는 방식이다. 저능력 모드의 파라미터를 기기가 직접 정할 수 있어 웨어러블부터 XR 헤드셋까지 각자 맞춤 설정이 된다. MLPM은 멀티링크 환경의 절전 시그널링을 정리한 것이다.

### P2P: TXSPG와 Co-CR

- **TXSPG:** AP가 자기 TXOP 일부를 P2P 그룹(예: 게임 콘솔과 컨트롤러 기기들)에 주기적으로 떼어준다. 그 시간엔 그룹 멤버만 전송한다.
- **Co-CR:** 이웃 AP들이 협의해서 P2P용으로 비워둘 채널을 일관되게 추천한다. 아파트 단위의 "P2P 전용 채널" 같은 개념이다.

## Multi-AP Coordination (MAPC)

MAPC는 Wi-Fi 8에서 처음으로 "AP끼리 MAC 레벨에서 실시간 협력"을 표준화한 기능이다. 지금까지 AP들은 같은 집 안에 있어도 서로를 남(OBSS)처럼 대하며 경쟁했다.

TGbn에서 논의·채택된 주요 협력 방식은 다음과 같다. 세부 동작은 아직 드래프트 단계라 바뀔 수 있다.

| 방식 | 핵심 아이디어 | 기대 효과 |
|---|---|---|
| Co-TDMA | 한 AP가 얻은 TXOP를 이웃 AP와 시간 분할로 나눠 씀 | 경쟁 충돌 감소, 지연 안정화 |
| Co-SR (Coordinated Spatial Reuse) | 이웃 AP가 송신 전력을 조절해 동시에 전송 | 같은 시간대 재사용으로 용량 증가 |
| Co-BF (Coordinated Beamforming) | 이웃 AP의 단말 방향으로 널(null)을 만들어 간섭 제거 | 동시 전송 시 간섭 억제 |
| Co-RTWT | 지연 민감 트래픽용 R-TWT 서비스 구간을 AP 간 맞춤 | 저지연 트래픽 보호 |

여러 AP가 같은 데이터를 함께 보내는 Joint Transmission은 연구 단계에서 많이 거론됐지만, AP 간 정밀 동기와 백홀 요구가 커서 1차 범위에서는 빠진 것으로 알려져 있다.

### EasyMesh와 뭐가 다른가

둘은 경쟁 관계가 아니라 층위가 다르다.

| 구분 | EasyMesh (IEEE 1905.1 기반) | 802.11bn MAPC |
|---|---|---|
| 계층 | 상위 제어 평면 | MAC (무선 구간) |
| 시간 단위 | 초~분 (설정, 스티어링, 채널 계획) | TXOP 단위 (ms 이하) |
| 전달 경로 | 백홀 (이더넷 또는 Wi-Fi) 위 1905 메시지 | 무선 관리·제어 프레임 |
| 대상 | 컨트롤러와 에이전트 | 이웃 AP 간 (같은 벤더·네트워크가 아니어도 가능) |

현실적인 그림은 EasyMesh 컨트롤러가 "어느 AP끼리 MAPC 그룹을 맺고 어떤 방식을 쓸지"를 정하고, 실제 TXOP 단위 협력은 AP 칩·펌웨어가 무선으로 처리하는 구조다. 메시 제품을 만드는 입장에서는 두 계층을 잇는 인터페이스가 새 개발 포인트가 된다. 반대로 MAPC는 서로 다른 집의 AP끼리도 협력할 수 있어서, 이웃 AP와 어디까지 정보를 주고받을지가 보안·프라이버시 쟁점이 될 수 있다.

## AP 개발자 관점: 지금부터 준비할 것

Wi-Fi 8 제품 경쟁은 "칩 나오면 포팅"으로는 늦다. Wi-Fi 7까지는 대부분 칩 기능을 켜는 일이었지만, Wi-Fi 8은 AP 소프트웨어가 정책을 결정해야 하는 기능이 많다.

- [ ] **측정 지표부터 바꾸기:** 최고 처리량 대신 95퍼센타일 지연, 엣지 처리량, 로밍 중 패킷 손실을 테스트 스펙의 기본 지표로 넣는다. PAR 목표가 곧 고객과 경쟁사가 비교할 지표다.
- [ ] **MLO 완성도 점검:** SMD, DPS, MAPC 모두 Wi-Fi 7 MLO 위에 올라간다. 지금 MLO 연결·링크 관리가 불안하면 Wi-Fi 8 기능은 시작도 못 한다.
- [ ] **멀티 AP 정책 엔진:** MAPC 그룹 구성, NPCA 채널 선택, DSO 서브밴드 배치, Co-CR 채널 추천은 결국 AP 소프트웨어의 판단이다. EasyMesh 컨트롤러 쪽에 이 정책을 담을 구조를 미리 설계한다.
- [ ] **SMD 보안 설계:** AP 간에 보안 컨텍스트가 넘어간다. SMD-ME를 어디에 둘지(컨트롤러 AP, 클라우드), 컨텍스트 전송 경로를 어떻게 보호할지가 핵심 보안 요구사항이 된다.
- [ ] **Draft 제품 리스크 관리:** 경쟁사가 Draft 기반 제품을 먼저 낸다. 우리도 따라갈지, 인증 시점에 맞출지 사업 판단이 필요하고, 따라간다면 펌웨어 업데이트로 최종 표준을 맞출 수 있는 구조인지 확인한다.
- [ ] **계측 장비 준비:** 이미 Wi-Fi 8 신호 분석·시그널링 테스터가 나오기 시작했다. 2027년 예산에 반영할 시점이다.

## 마치며

Wi-Fi 8은 스펙 시트의 숫자로는 Wi-Fi 7과 거의 같아 보인다. 하지만 실제로는 AP가 혼자 잘하는 시대에서 AP끼리 협력하고 단말과 더 촘촘히 조율하는 시대로 넘어가는 전환점이다.

다음 편에서는 이 글에서 짧게 다룬 MAPC를 EasyMesh 구조와 나란히 놓고 더 깊게 파볼 예정이다.

### 참고자료

- [Samsung Research: IEEE 802.11bn (UHR, Wi-Fi 8)](https://research.samsung.com/blog/IEEE-802-11bn-Ultra-High-Reliability-UHR-Wi-Fi-8) — PAR 목표, ELR, DRU, P-EDCA, LLI, NPCA, DSO, SMD, DPS, P2P
- [Wi-Fi NOW: Wi-Fi 8 standard on track for completion in 1H2028](https://wifinowglobal.com/news-blog/wi-fi-8-standard-on-track-for-completion-in-1h2028-with-certification-scheduled-for-january-2028/) — 승인·인증 일정
- [ABI Research: Wi-Fi 8 표준화 진행 현황](https://www.abiresearch.com/market-research/insight/7788568-wi-fi-8-presents-both-lucrative-opportunit?hslang=en) — D2.0, 이후 일정
- [Wikipedia: Wi-Fi 8](https://en.wikipedia.org/wiki/Wi-Fi_8) — 드래프트 이력
- [Inquirer Technology: TP-Link Archer 8](https://technology.inquirer.net/147067/tp-link-announces-archer-8-its-first-wi-fi-8-router-platform-designed-for-real-world-reliability) — 첫 Wi-Fi 8 라우터 플랫폼
- [What Will Wi-Fi 8 Be? A Primer on IEEE 802.11bn UHR (arXiv)](https://arxiv.org/abs/2303.10442v2) — AP 협력 배경
