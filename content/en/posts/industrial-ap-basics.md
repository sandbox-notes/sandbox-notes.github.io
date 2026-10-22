---
title: "How Is an Industrial AP Different?"
date: 2026-11-04T09:00:00+09:00
draft: false
series: ["Industrial Wireless"]
tags: ["industrial wireless", "Industrial Wi-Fi", "AP", "IEC 62443", "AGV", "OT security"]
description: "Why an industrial AP differs on almost every line of the spec sheet even though it uses the same 802.11 chip: radio, hardware, certification, software and security."
---

> This post summarizes public standards and common industry requirements. Figures differ from product to product.

## Introduction

The core requirement for an industrial AP is not "fast" but "never stops." When a home AP drops, video stutters. When a factory AP drops, the line stops and robots perform emergency stops.

From the outside, an industrial AP uses the same 802.11 chip and runs the same hostapd. But once you receive the requirements from an industrial site, almost every line of the spec sheet changes: operating temperature, power, mounting, certification, firmware support period, even roaming time requirements.

This series starts by organizing the differences a developer who has built ordinary Wi-Fi APs runs into when first looking at industrial wireless. This first installment is an introductory map of the whole territory.

## Side-by-side comparison

The biggest difference is the optimization target. Home and enterprise APs put throughput and concurrent client count first. Industrial APs put predictable latency and availability first.

| Item | General (home / enterprise) AP | Industrial AP |
|---|---|---|
| Key metrics | Peak throughput, concurrent clients | Worst-case latency, drop during roaming, uptime |
| Operating temperature | Roughly 0 to 40 °C | Typically -40 to 70 °C or more |
| Enclosure | Plastic, indoor | Metal, IP30 (cabinet) to IP67/68 (exposed field) |
| Mounting | Ceiling, wall | DIN rail, pole, vehicle or robot mount |
| Power | Single adapter or PoE | Redundant DC inputs (for example 12 to 48 VDC) + PoE |
| Antenna | Internal | External connectors (N-type, RP-SMA), choice of directional antennas |
| Roaming | User-perceived level | Millisecond-level requirements for mobile robots and vehicles |
| Certification | Radio approval, safety | + Industrial EMC, vibration and shock, explosion protection, rail, industrial-protocol conformance |
| Lifecycle | New model every 2 to 3 years | Expected supply and support for 10+ years |
| Management | Cloud controller, app | SNMP, industrial network management tools, field-engineer access |

The figures vary by product, but the direction is the same across nearly every industrial product line.

## Radio: look at the worst case, not the average

An average latency of 5 ms means nothing in industrial wireless. If PLC communication does not get a reply within its fixed cycle, it is treated as a communication fault and the equipment stops, so what matters is the slowest packet.

### Determinism

Wi-Fi is contention-based (CSMA/CA), so latency is probabilistic. Industrial products compensate for this limit in a few ways.

- **Vendor-specific scheduling:** the AP directly controls client transmissions with polling or time slots. Siemens iPCF is the classic example. The catch is that it only works well between devices from the same vendor.
- **Traffic separation:** control traffic is isolated onto a dedicated SSID, dedicated channel and high QoS, and not mixed with video or office traffic.
- **Using standard features:** Wi-Fi 6's OFDMA and TWT, Wi-Fi 7's R-TWT, and Wi-Fi 8's P-EDCA and MAPC are moving, step by step, toward raising determinism inside the standard.

### AGV/AMR roaming

Mobile robots are the most demanding customers in industrial wireless. They keep moving between APs at several km/h, and a drop of just a few hundred milliseconds during roaming can trigger a safety stop.

The way an ordinary client behaves, holding on until the signal weakens and only then scanning, will not work. So industrial products combine scanning only a preselected set of channels quickly, choosing the next AP in advance based on direction of travel, and shortening reauthentication with 802.11r. Moxa Turbo Roaming is one example of a vendor packaging this as a product feature. If Wi-Fi 8's SMD (make-before-break roaming) is standardized, this area could change substantially.

### Interference environment

A factory is the worst environment there is for radio. Metal structures and moving equipment keep changing the multipath, and inverters, welders and motors emit broadband noise. 2.4 GHz is often saturated by other wireless devices, so control traffic is moving to 5 GHz or 6 GHz. A site survey before installation and a fixed channel plan are essential.

## Hardware: same chip, different body

For an industrial AP, cost and development time are decided less by the chipset than by the hardware design around it. Simply putting a general AP platform into a metal case will, in most cases, fail certification.

- **Wide-temperature operation:** to operate from -40 to 70 °C you have to move to industrial-grade parts from the start, and dissipate heat without a fan. For a hot SoC such as a Wi-Fi 7 chip, surviving high temperature without throttling is the biggest hurdle.
- **Dust and water protection:** IP30 is typical for inside a cabinet, and exposed field units go up to IP67/68. Connectors and cable glands must meet the same rating.
- **Vibration and shock:** on a robot or vehicle, connector retention, component fixing and solder reliability become issues. Screw-locking connectors (such as M12) are widely used.
- **Power redundancy:** 24 VDC is the standard in factory control panels. Units accept two wide-range DC inputs so that one failing does not stop operation, often alongside PoE. Reverse-polarity and surge protection are the baseline.
- **External antennas:** the antennas must be swappable between directional and omnidirectional to suit the installation. For coverage along an aisle or a rail, a directional antenna is decisive.
- **Mounting:** DIN rail, pole, wall and vehicle brackets, that is, the standard mounting methods of the field, must be supported.

## Certification and standards: the real barrier to entry

The real barrier to entry in the industrial AP market is certification, not technology. Customers filter first by asking "do you have this certification?", and each certification takes months and considerable cost.

| Category | Representative standard | What it checks | Market that needs it |
|---|---|---|---|
| Industrial EMC | IEC 61000-6-2 (immunity), 61000-6-4 (emissions) | No malfunction under factory-level electrical noise | General factory |
| Environmental testing | IEC 60068-2 series (vibration -6, shock -27, etc.) | Durability against vibration, shock, temperature and humidity | Factory, mobile equipment |
| Explosion protection | ATEX, IECEx, UL Class I Div 2 | Not an ignition source in explosive atmospheres | Petrochemical, gas |
| Rail | EN 50155, EN 50121 | Requirements for on-board electronics | Rail |
| Security | IEC 62443-4-1, 4-2 | Secure development process, component security functions | Factories and infrastructure generally |
| Industrial protocol conformance | PROFINET, EtherNet/IP, CC-Link and others, certified by each body | Interoperability on that industrial network | The relevant PLC ecosystem |

The last row matters most. Industrial sites are divided by PLC vendor ecosystem: PROFINET is strong in Europe, EtherNet/IP in North America, and the CC-Link family in Japan and parts of Asia. Which ecosystem you target completely changes the certifications you need and the partners you need. That gets a deeper treatment in the next installment.

Radio approvals (KC, FCC, CE RED and so on) are the same as for a general AP, but remember that making the external antenna replaceable widens the scope of certification for each antenna combination.

## Software and operations: firmware that lasts ten years

Industrial customers use a piece of equipment for ten years or more once it is installed. So the software requirement is less about "new features" and more about "long-lived, unchanging, and fixable."

- **Long-term support branches:** you must maintain, for years, an LTS firmware that takes only security patches, separate from the feature branch. Because product support often outlasts the chip vendor's SDK support, you need the capability to backport kernel and hostapd security patches yourself.
- **Behavioral consistency:** if a firmware upgrade subtly changes roaming timing or QoS behavior, that is an outage in the field. Regression tests need quantitative metrics such as latency and roaming time.
- **Wired redundancy:** for an industrial AP the wired uplink matters as much as the radio. Support for ring redundancy such as MRP (IEC 62439-2) and for vendor-specific ring protocols is a line item in spec comparisons.
- **Field management:** air-gapped networks with no cloud connection are the default. You need SNMP, a local web UI, CLI, configuration backup and restore, and integration with industrial network management tools. A field engineer must be able to replace and recover a unit with just a laptop.
- **Client mode:** industrial wireless has as much demand for client (bridge) mode as for AP mode. It attaches to a machine or robot and connects wired equipment wirelessly. It is common for the same hardware to support AP, client and bridge.

## Security: availability comes before confidentiality

IT security looks at confidentiality first, but industrial (OT) security looks at availability first. The line stopping is a bigger incident than data leaking.

From that viewpoint, the most realistic threats to industrial wireless are:

- **Denial of service:** forged deauthentication frames, jamming, channel saturation. Radio can be attacked from outside the factory fence. Mandatory PMF (802.11w) is the minimum condition.
- **Weak authentication:** PSKs that have not been changed for years for the sake of field convenience, or old settings such as WPA2-TKIP kept for legacy devices, are common.
- **Flat networks:** a structure where breaking into one wireless link reaches the PLC directly. The wireless segment should be separated into its own zone, with conduits restricted by a firewall.
- **Exposed management interfaces:** default accounts, plaintext management protocols and old firmware.

IEC 62443 is the family of industrial security standards that systematizes these problems. For an equipment maker two parts are key. 62443-4-1 defines the secure development lifecycle (threat modeling, security testing, vulnerability response process), and 62443-4-2 defines the security capabilities a component must have (authentication, access control, integrity, logging and so on) by security level (SL).

For a general AP company, 62443 certification is not simply adding features but changing the whole development process. In exchange, once you have it, it is a strong differentiator in the industrial market.

## Checkpoints when a general AP developer moves to industrial

Your wireless expertise carries over unchanged. What has to change are the test metrics, the support system, and the decision about "whose ecosystem to attach to."

- [ ] **Decide the target market and ecosystem first:** which industry (manufacturing, logistics, rail, energy) and which PLC ecosystem you target determines the certification list and the partners. This comes before technical development.
- [ ] **Redefine the test spec:** use 99.9th-percentile latency, roaming interruption time and weeks-long continuous operation as the primary metrics instead of throughput.
- [ ] **Reconsider the hardware platform:** can the existing platform handle wide temperature, fanless cooling and dual DC input? If not, you need an industrial-specific board.
- [ ] **Secure roaming performance:** measure 802.11r, channel-restricted scanning and fast handoff, including in client mode. Build a mobile test environment that can reproduce AGV scenarios.
- [ ] **Certification roadmap and budget:** set the schedule and cost in advance for the order in which you will obtain industrial EMC, environmental testing, protocol conformance and 62443.
- [ ] **Long-term support system:** a plan for running LTS branches, backporting security patches and handling component end-of-life.

## Closing

An industrial AP is not a "tough AP" but a different product designed for a different goal. Even with the same chip, the metrics, hardware, certification and support system all change.

The next installment covers what is hard about carrying industrial Ethernet protocols (the PROFINET, EtherNet/IP and CC-Link families) over wireless, and what each ecosystem requires of wireless equipment.

### Standards to consult

- IEC 61000-6-2 / 6-4: EMC for industrial environments
- IEC 60068-2 series: environmental testing (vibration, shock, temperature and humidity)
- IEC 62439-2: MRP (ring redundancy)
- IEC 62443-4-1 / 4-2: industrial control system security (development process, component requirements)
- EN 50155 / EN 50121: rail rolling-stock electronics
- ATEX / IECEx: explosion protection
