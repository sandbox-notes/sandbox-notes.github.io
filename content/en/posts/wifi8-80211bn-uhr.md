---
title: "Wi-Fi 8 (802.11bn UHR): It Is About Reliability, Not Speed"
date: 2026-11-11T09:00:00+09:00
draft: false
series: ["New Wi-Fi Specs"]
tags: ["Wi-Fi 8", "802.11bn", "UHR", "MAPC", "EasyMesh", "AP"]
description: "The goals and key technologies of 802.11bn (UHR), the first Wi-Fi generation that does not raise peak speed, and what AP developers should prepare now."
---

> This post is based on public sources. 802.11bn is still at the draft stage, so details and schedules may change, and sources disagree slightly on dates. As of October 2026.

## Introduction

Wi-Fi 8 (IEEE 802.11bn) is the first generation in Wi-Fi history that does not raise peak speed. Even the name says so: UHR, Ultra High Reliability.

From Wi-Fi 4 to 7, speed grew by widening bandwidth, raising modulation order and adding streams. But with Wi-Fi 7 reaching 320 MHz, 4096-QAM and 16 streams, there is less room left to raise the "peak speed in an ideal environment," and less that users would feel. The problems real users hit are speed falling off a cliff at the edge of coverage, interference from a neighbor's AP, dropouts during roaming, and latency spikes in games and video calls.

Wi-Fi 8 tackles exactly this "performance in bad conditions" head-on. And since one of its core solutions is cooperation between APs, Multi-AP Coordination, it has a good chance of being the most important generation for anyone building mesh and multi-AP products.

## The goals: UHR in numbers

The 802.11bn PAR set, instead of peak speed, felt-experience targets of "25% versus Wi-Fi 7." The basic skeleton (2.4/5/6 GHz, up to 320 MHz, 4096-QAM) is the same as Wi-Fi 7.

| PAR target | Meaning | Technologies that address it |
|---|---|---|
| 25% or more throughput gain in poor signal conditions | Coverage edge, obstacle-heavy environments | ELR, DRU |
| 25% lower 95th-percentile latency | Reduce "spiking latency," not the average | P-EDCA, LLI, NPCA, DSO |
| 25% less packet loss during roaming | Dropouts while moving between APs | SMD (Seamless Mobility Domain) |
| Lower power consumption | Wearables, mobile APs | DPS, MLPM |
| Better P2P operation | Casting, XR, local gaming | TXSPG, Co-CR |

Note that the latency target is the **95th percentile**, not the average. What irritates users is not an average of 10 ms but an occasional 200 ms spike, and Wi-Fi 8 aims to cut that tail.

## The standardization schedule

As of October 2026, 802.11bn is known to be at the Draft 2.0 ballot stage. Sources disagree on when the ballot ran, giving either around July or early August to early September. Final IEEE approval is targeted for the first half of 2028, and Wi-Fi Alliance certification for late 2027 to January 2028.

| When | What |
|---|---|
| 2023-11 | TGbn standardization work begins |
| 2025-07 | Draft 1.0 completed (technical scope fixed). Some sources report more than 7,000 comments in the first ballot |
| Mid-2026 | Draft 2.0 ballot (sources differ: July to September) |
| Mid-2027 | Wi-Fi Alliance certification test plan expected to be finalized |
| 2027-12 to 2028-01 | Target start of Wi-Fi Alliance Wi-Fi 8 certification |
| First half of 2028 | Target for final IEEE 802.11bn approval (March to May depending on source; Wikipedia projects September) |

The market is already ahead. TP-Link unveiled its Archer 8 platform, announced for an October 2026 launch, at IFA 2026 as the Archer 8 Ultra BN19000 (19 Gbps class, manufacturer figure), and the Deco 8 mesh system is expected in the first half of 2027 (Q1 or Q2 depending on the source). Reports also say US availability is uncertain because of regulatory issues. These are draft-based products, so, as with Wi-Fi 6 and 7, the gap to the final standard and certification will have to be closed with firmware updates.

## PHY technologies: saving the coverage edge

The PHY changes focus on rescuing "distant clients and clients with weak uplink." The representatives are ELR and DRU.

### ELR (Enhanced Long Range)

A long-range PPDU format that keeps working even at low speed, aimed at IoT sensors and clients with weak transmit power.

- After the legacy preamble it adds two mark symbols that signal ELR mode, and it boosts the UHR-STF/LTF by 3 dB so that the frame can be picked up even at low SNR.
- It uses only 20 MHz, 1 stream and BPSK/QPSK (MCS0/1), with PHY rates of about 1.67 / 3.33 Mbps.
- It repeats a 52-tone RU four times within 20 MHz for a gain of about 6 dB.

### DRU (Distributed Resource Unit)

An idea to solve the uplink link-budget problem within regulatory limits. In an environment with a PSD (power per MHz) limit, such as 6 GHz LPI, sending on a small RU caps total transmit power low.

DRU scatters the tones of a small RU across the whole 40/80 MHz instead of grouping them contiguously. It keeps to the per-MHz power limit while raising total power, and according to the literature this gives up to about 11 dB of power gain. In uplink OFDMA, several clients can use different DRUs at the same time.

### Summary

Neither ELR nor DRU contributes anything to the "peak speed" metric. Instead they are technologies that rescue the back bedroom that used to need one more AP. From the standpoint of mesh AP sales volume, that could be a double-edged sword.

## MAC technologies: latency tails, wasted wide channels, roaming dropouts

Most of Wi-Fi 8's real change is in the MAC. The core is solving three problems: priority traffic occasionally waiting a long time, a single primary 20 MHz blocking an entire wide channel, and dropping during roaming.

### Low latency: P-EDCA and LLI

- **P-EDCA:** adds another "express lane" on top of the existing EDCA. When the previous TXOP ends, a P-EDCA device does not wait out the full AIFS but grabs the medium first with a CTS frame called DS-CTS, then contends only among its peers. The goal is to cut the tail of the latency distribution, not to raise average throughput, and the AP controls abuse through parameters.
- **LLI (Low Latency Indication):** a device responding inside another device's TXOP signals "I have urgent data queued" with a single bit in the response frame. The AP sees it and grants an uplink slot quickly.

### Using wide channels properly: NPCA and DSO

- **NPCA (Non-Primary Channel Access):** when the primary 20 MHz is busy because of a neighboring BSS, the whole 160/320 MHz channel has so far sat idle. NPCA lets a transmission use another preselected 20 MHz as a temporary primary. The effect is large in OBSS-dense environments such as apartments.
- **DSO (Dynamic Sub-band Operation):** if the AP is 320 MHz but a client supports only 80 MHz, until now all such clients crowded onto the same 80 MHz. With DSO the AP moves clients onto different 80 MHz slices of the 320 MHz and serves them in parallel. The switch is done with an ICF/ICR frame exchange.

### Seamless roaming: SMD

SMD (Seamless Mobility Domain) groups several APs into one logical domain, with a management entity (SMD-ME) holding the association and authentication state in place of the APs. On roaming it hands the MAC state, QoS parameters and security context to the target AP, and uses MLO to attach the new link first and drop the old one afterwards (make-before-break).

1. **Prepare:** the current AP passes the context to the target AP through the SMD-ME, and sets up a link to the target AP in advance in a power-save state.
2. **Execute:** the client activates the new link without reauthenticating.
3. **Complete:** once the connection is stable, the previous AP cleans up the temporary state.

If 802.11r (FT) was an approach that "shortens reconnection time," SMD is an approach that "removes reconnection altogether." From a security standpoint, the fact that the security context moves between APs is a new attack surface.

### Power saving: DPS and MLPM

DPS (Dynamic Power Save) lets a client or mobile AP idle at low capability (narrow bandwidth, few streams, low MCS) and switch to high-performance mode when it receives an ICF. Because the device can set the low-capability parameters itself, everything from wearables to XR headsets can be tuned individually. MLPM tidies up power-save signaling in a multi-link setting.

### P2P: TXSPG and Co-CR

- **TXSPG:** the AP periodically hands part of its own TXOP to a P2P group (for example a game console and its controllers). During that time only group members transmit.
- **Co-CR:** neighboring APs negotiate and consistently recommend which channel to leave free for P2P. Think of a building-wide "P2P-only channel."

## Multi-AP Coordination (MAPC)

MAPC is the first feature in Wi-Fi 8 to standardize "real-time cooperation between APs at the MAC level." Until now, APs treated each other as outsiders (OBSS) and competed even when they were in the same home.

The main cooperation methods discussed and adopted in TGbn are below. The details are still at the draft stage and may change.

| Method | Core idea | Expected effect |
|---|---|---|
| Co-TDMA | One AP shares the TXOP it won with a neighboring AP by time division | Fewer contention collisions, steadier latency |
| Co-SR (Coordinated Spatial Reuse) | Neighboring APs adjust transmit power and transmit at the same time | More capacity by reusing the same time |
| Co-BF (Coordinated Beamforming) | Form nulls toward a neighboring AP's clients to cancel interference | Interference suppression during simultaneous transmission |
| Co-RTWT | Align R-TWT service periods for latency-sensitive traffic across APs | Protects low-latency traffic |

Joint Transmission, where several APs send the same data together, was widely discussed at the research stage, but it is reportedly left out of the first scope because of the precise inter-AP synchronization and backhaul it needs.

### How is it different from EasyMesh?

The two are not competitors; they sit at different layers.

| Aspect | EasyMesh (based on IEEE 1905.1) | 802.11bn MAPC |
|---|---|---|
| Layer | Upper control plane | MAC (radio segment) |
| Time scale | Seconds to minutes (configuration, steering, channel planning) | Per TXOP (sub-millisecond) |
| Transport | 1905 messages over the backhaul (Ethernet or Wi-Fi) | Wireless management and control frames |
| Participants | Controller and agents | Between neighboring APs (even if not from the same vendor or network) |

The realistic picture is that an EasyMesh controller decides "which APs form a MAPC group and which method they use," while the actual TXOP-level cooperation is handled over the air by the AP's chip and firmware. For anyone building a mesh product, the interface that connects the two layers becomes a new development item. Conversely, MAPC can also work between APs in different homes, so how much information to exchange with a neighboring AP can become a security and privacy question.

## The AP developer's view: what to prepare now

Wi-Fi 8 product competition will not be won by "porting once the chip arrives." Up to Wi-Fi 7 it was mostly a matter of switching on chip features, but Wi-Fi 8 has many features where AP software has to make policy decisions.

- [ ] **Change the metrics first:** make 95th-percentile latency, edge throughput and packet loss during roaming the primary metrics in the test spec instead of peak throughput. The PAR targets are what customers and competitors will compare.
- [ ] **Check how complete your MLO is:** SMD, DPS and MAPC all sit on top of Wi-Fi 7 MLO. If MLO connection and link management are shaky today, Wi-Fi 8 features cannot even get started.
- [ ] **A multi-AP policy engine:** MAPC group formation, NPCA channel selection, DSO sub-band placement and Co-CR channel recommendation are ultimately AP software decisions. Design in advance a structure on the EasyMesh controller side to hold this policy.
- [ ] **SMD security design:** security context is handed between APs. Where to put the SMD-ME (a controller AP, the cloud) and how to protect the context transfer path become key security requirements.
- [ ] **Manage draft-product risk:** competitors will ship draft-based products first. A business decision is needed on whether to follow or to time the launch to certification, and if you follow, confirm that the product can be brought to the final standard through a firmware update.
- [ ] **Prepare test equipment:** Wi-Fi 8 signal analyzers and signaling testers are already starting to appear. It is time to put them in the 2027 budget.

## Closing

On the numbers in a spec sheet, Wi-Fi 8 looks almost the same as Wi-Fi 7. In practice, though, it is the turning point from an era in which an AP does well on its own to one in which APs cooperate and coordinate more tightly with clients.

The next installment will take MAPC, touched on only briefly here, and look at it more deeply alongside the EasyMesh architecture.

### References

- [Samsung Research: IEEE 802.11bn (UHR, Wi-Fi 8)](https://research.samsung.com/blog/IEEE-802-11bn-Ultra-High-Reliability-UHR-Wi-Fi-8): PAR targets, ELR, DRU, P-EDCA, LLI, NPCA, DSO, SMD, DPS, P2P
- [Wi-Fi NOW: Wi-Fi 8 standard on track for completion in 1H2028](https://wifinowglobal.com/news-blog/wi-fi-8-standard-on-track-for-completion-in-1h2028-with-certification-scheduled-for-january-2028/): approval and certification schedule
- [ABI Research: Wi-Fi 8 standardization progress](https://www.abiresearch.com/market-research/insight/7788568-wi-fi-8-presents-both-lucrative-opportunit?hslang=en): D2.0 and later schedule
- [Wikipedia: Wi-Fi 8](https://en.wikipedia.org/wiki/Wi-Fi_8): draft history
- [Inquirer Technology: TP-Link Archer 8](https://technology.inquirer.net/147067/tp-link-announces-archer-8-its-first-wi-fi-8-router-platform-designed-for-real-world-reliability): the first Wi-Fi 8 router platform
- [What Will Wi-Fi 8 Be? A Primer on IEEE 802.11bn UHR (arXiv)](https://arxiv.org/abs/2303.10442v2): background on AP cooperation
