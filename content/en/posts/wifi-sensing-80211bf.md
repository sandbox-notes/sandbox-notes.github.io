---
title: "Wi-Fi Sensing: IEEE 802.11bf Turns Wi-Fi into a Sensor"
date: 2026-10-28T09:00:00+09:00
draft: false
series: ["New Wi-Fi Specs"]
tags: ["Wi-Fi", "802.11bf", "Wi-Fi Sensing", "CSI", "privacy", "AP"]
description: "How 802.11bf, the first official standard for detecting motion and presence with ordinary Wi-Fi signals, works, where the market stands, and what it means for privacy."
---

> This post is based on public standards documents and public reporting. Product and schedule details are as of October 2026.

## Introduction

IEEE 802.11bf, approved on 28 May 2025 and published on 26 September 2025, is the first official standard for using Wi-Fi as a sensor. Technology that detects a person's movement, whether a room is occupied, and even breathing, using the same radio signals used for communication, has finally been brought into the standard.

Wi-Fi sensing itself is not new. Academia has studied CSI-based sensing for more than ten years, and some ISPs already sell proprietary services such as "WiFi Motion." What has changed is that chipsets, APs and client devices can now exchange measurements **using the same procedures**.

I picked 802.11bf to open the new-specs series because it is the most direct opportunity, and the most direct homework, for an AP maker. This post covers the principle, the standard's structure, the market, and, as befits a security series, the privacy issues.

## The principle: radio waves "see" people

The raw material of Wi-Fi sensing is CSI (Channel State Information). When a person moves, the multipath environment changes, and that change is imprinted on the CSI as changes in per-subcarrier amplitude and phase.

Indoors, the signal between a transmitter and a receiver does not take a single straight path. It is reflected and absorbed by walls, furniture and bodies, and spreads over dozens of paths. An OFDM receiver estimates this channel for each subcarrier in order to demodulate, and that estimate is the CSI. It was originally an internal value for the equalizer, but stacked as a time series it becomes sensor data that reads changes in the environment.

- **Presence and motion detection:** if CSI variance grows, someone is moving. This is the simplest use and the most commercialized.
- **Location and direction:** estimate the angle of arrival (AoA) from phase differences across several links or an antenna array.
- **Breathing and heartbeat:** millimeter-scale chest motion creates periodic phase changes. Separate out the Doppler component and you can extract at least breaths per minute.
- **Gesture and activity recognition:** classify time-frequency patterns with machine learning.

Wider bandwidth improves time resolution, and more antennas improve spatial resolution. That is why Wi-Fi 7's 320 MHz channels and multiple antennas matter for sensing too. The 60 GHz band has short wavelengths and allows radar-like precision.

## The structure of 802.11bf

802.11bf did not create a new PHY. It is a MAC-centered amendment that layers "procedures for requesting and reporting sensing measurements" on top of the existing HE/EHT (Wi-Fi 6/7) and DMG/EDMG (60 GHz) PHYs. It applies to 1 to 7.125 GHz and above 45 GHz.

### Roles

- **Sensing initiator / responder:** the party that requests sensing and the party that responds. Either an AP or a STA can be either one.
- **Sensing transmitter / receiver:** the party that actually sends the measurement frames (NDPs) and the party that receives them and measures CSI.

The point is that the requesting roles and the transmit/receive roles are separate. For example, if an AP is both initiator and receiver, the AP ends up measuring signals sent by several STAs.

### The sensing procedure (sub-7 GHz)

1. **Sensing session setup:** both sides exchange sensing capabilities and open a session.
2. **Sensing measurement setup:** they agree on measurement parameters (bandwidth, report format and so on) and assign an ID.
3. **Sensing measurement instance:** the actual measurements are performed repeatedly.
4. **Sensing measurement setup termination:** the setup is torn down.

There are two kinds of measurement instance.

- **TB (Trigger-Based) sensing:** the AP is the initiator. It calls several STAs at once with a Trigger frame, runs NDP sounding, and collects reports. It groups many devices efficiently, which suits AP-centric services.
- **Non-TB sensing:** the STA is the initiator and the AP the responder. Used when the device drives the measurement.

### SBP (Sensing By Proxy)

A feature that lets a non-AP STA with little or no sensing capability ask the AP: "sense for me and just give me the result." It assumes scenarios such as a smartphone app consuming the AP's sensing results.

### DMG sensing (60 GHz)

It defines radar-like modes such as monostatic (transmit and receive by yourself), bistatic, multistatic and coordinated monostatic. Being beamforming-based, it is precise but has short coverage.

## Standardization status

The standard is done; what remains is chipset support and a certification program. As of October 2026, no Wi-Fi Alliance Wi-Fi sensing certification program has been confirmed.

| When | What |
|---|---|
| 2025-09-26 | IEEE Std 802.11bf-2025 published (802.11 Amendment 4, about 228 pages) |
| 2025-05-28 | Approved by the IEEE SA Standards Board (based on D8.0) |
| 2023 | First draft released |
| 2020 | Task Group 802.11bf begins work |
| 2018 onward | The Wireless Broadband Alliance (WBA) publishes a Wi-Fi sensing white paper and drives the standardization discussion |

As a document, 802.11bf is an amendment to a version of 802.11-2024 that already includes 802.11be (Wi-Fi 7) and others. That means it is landing in Wi-Fi 7-generation chips and later, and major chip vendors such as MediaTek and Qualcomm are reportedly working to include 802.11bf in chips from Wi-Fi 7 onward.

The WBA separately publishes Wi-Fi sensing test methodology and deployment guideline documents, and has opened a work stream to build the test plan that would underpin a certification program.

## Use cases and commercial products

The commercial market opened before the standard. Most services on sale today use proprietary technology that predates 802.11bf, and the dominant model is an ISP adding software to a home gateway.

| Company | Form | Notes |
|---|---|---|
| Comcast (Xfinity WiFi Motion) | ISP service | Rolled out in earnest in August 2026 as part of Xfinity Shield (some reports say some users saw it earlier). Detects motion from signal changes between the gateway and connected devices, with Home, Away and Dark Watch modes |
| Cognitive Systems (WiFi Motion) | Licensed software | Built into Broadcom Wi-Fi 6/6E AP chips, also working with Qualcomm and Marvell. Adopted by many ISPs |
| Origin Wireless | Licensed software | Focused on presence and wellness |

The main uses are:

- **Home security:** motion alerts while away. The first area to make money.
- **Elder care:** monitoring daily patterns, long-inactivity alerts, fall detection.
- **Energy management:** controlling lighting and HVAC by occupancy, extending to space-utilization analytics in commercial buildings.
- **Network diagnostics:** reusing sensing data for Wi-Fi quality analysis.

The biggest selling point is that it can cover bathrooms and bedrooms where cameras cannot go. And that leads straight into the privacy problem in a later section.

## The AP developer's view

For an AP, 802.11bf is not "done once the chip supports it." It means building a new pipeline that runs through firmware, drivers, user space and the cloud. In particular, you have to settle early in the design that sensing and communication share the same airtime.

- [ ] **Check the chipset's support scope:** confirm in the vendor SDK whether firmware handles the 802.11bf procedures, or only exposes CSI and leaves the procedures to the host.
- [ ] **Design the CSI extraction path:** driver → netlink/debugfs → user-space daemon → analytics engine. With a short measurement period the CPU and memory load is larger than you expect.
- [ ] **Measure the airtime impact:** quantify how much TB sensing's sounding and reporting overhead affects throughput and latency. The first complaint will be "VoIP drops when sensing is on."
- [ ] **Multi-AP environments:** in a mesh, which AP leads sensing and how are results merged? Integration with multi-AP frameworks such as EasyMesh is still unsettled.
- [ ] **Relation to MLO:** on a Wi-Fi 7 MLD you need a policy on which link to sense over and how to treat per-link results.
- [ ] **Default policy:** whether sensing is off or on by default, and who may turn it on. This is a technical question and also the privacy question in the next section.

## Security and privacy issues

The biggest risk of Wi-Fi sensing is that sensors that can see through walls are already installed in every home. You can cover a camera lens, but radio waves are hard even to notice.

- **Passive eavesdropping:** CSI can be obtained by receiving alone. In research, attackers outside a home have estimated occupancy, movement and even keystroke patterns from Wi-Fi signals alone. The threat predates 802.11bf and does not go away because a standard exists.
- **The measurement data is itself sensitive:** breathing, sleep and daily patterns are close to health information. If that data goes from the AP to an ISP cloud, collection consent, retention and third-party disclosure follow. In Comcast's case, it says it does not identify people or provide video, but reports that it could hand over time-stamped motion data in response to legal requests such as warrants caused controversy.
- **Abuse of sensing requests:** if a connected device can ask the AP to measure through SBP or non-TB sensing, a scenario appears where another person on the same network (a roommate, a guest) obtains sensing results. Sensing permissions need to be separated from guest networks.
- **Forged measurement reports:** if security alerts or automations run on sensing results, you also have to consider attacks that forge or replay report frames to make it look like "nobody is there." The implementation must confirm that reports travel over a protected-frame path such as PMF.

In short, implementing 802.11bf is feature development and, at the same time, building a personal-data processing system. Default off, explicit user consent, separation of sensing permissions by role, and local processing first are the minimum principles.

## Closing

With 802.11bf, Wi-Fi became a sensor network as well as a communication network. For AP makers it is a new service opportunity, but it arrives with the homework of airtime management and privacy design.

The next new-specs installment covers **802.11bn (UHR)**, the basis of Wi-Fi 8.

### References

- [IEEE Std 802.11bf-2025 (standard document PDF)](https://people.freebsd.org/~adrian/net80211/80211bf-2025.pdf): approval and publication dates
- [ANSI Webstore: IEEE 802.11bf-2025](https://webstore.ansi.org/standards/ieee/ieee80211bf2025): applicable PHYs and bands
- [An Overview on IEEE 802.11bf: WLAN Sensing (arXiv)](https://arxiv.org/pdf/2207.04859): procedures, DMG sensing, technical candidates
- [NIST: IEEE 802.11bf: Enabling the Widespread Adoption of Wi-Fi Sensing](https://www.nist.gov/node/1742991)
- [WBA Wi-Fi Sensing project](https://wballiance.com/wi-fi-sensing/)
- [TechSpot: Comcast adds motion sensing to Xfinity routers (2026)](https://www.techspot.com/news/113538-comcast-adds-home-motion-sensing-millions-xfinity-routers.html)
- [The Next Web: Comcast Xfinity Shield WiFi Motion and privacy](https://thenextweb.com/news/comcast-xfinity-shield-wifi-motion-privacy)
- [Lightwave: Cognitive Systems brings motion sensing to Broadcom Wi-Fi 6/6E APs](https://www.lightwaveonline.com/broadband/article/55031708/cognitive-systems-brings-motion-sensing-to-broadcoms-wi-fi-6-6e-access-points)
