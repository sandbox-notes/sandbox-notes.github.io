---
title: "Network Signals in 287 Public Ransomware PCAPs (2017)"
date: 2026-10-05
draft: false
tags: ["ransomware", "pcap", "network-forensics", "malware-traffic-analysis", "Cerber", "Locky"]
description: "I counted which network signals show up, and how often, across 287 ransomware infection captures published in 2017. Every one of them was a single-host infection chain."
---

> The numbers here are re-tallied from the output table of a feature-extraction script I wrote. No pcaps or samples are shared.

## TL;DR

Across 287 public ransomware pcaps from 2017 there was **no sign of internal spread (SMB, RDP or ARP scanning)**. Every capture shows "one PC getting infected". The most common signals were **random-looking URIs (40%)**, **DGA-looking domains (29%)** and **Cerber-style UDP bursts (25%)**.

## Data

- Source: attachments to the 2017 posts on malware-traffic-analysis.net, 130 archives, **287 unique pcaps** after deduplication.
- Labels: family names taken from file names and post titles, so they are not authoritative.
- Packet counts: median 422, maximum 69,298 (a few pcaps are almost empty).

| Family | pcaps |
|---|---:|
| Cerber | 138 |
| Other / unlabeled | 50 |
| Locky | 31 |
| GlobeImposter | 20 |
| Spora | 12 |
| Jaff | 10 |
| CryptoMix | 7 |
| Sage | 7 |

## Method

- A script reads each pcap with `tshark` and **counts features**. Response bodies are streamed, never stored.
- Each signal is counted as "seen at least once in that pcap". These are **observation frequencies, not detection criteria**.

## Result 1: no internal spread

| Item | pcaps |
|---|---:|
| SMB target scanning | 0 |
| RDP target scanning | 0 |
| ARP sweep | 0 |
| Flagged for internal spread | **0 / 287** |

Every capture is a **single infection chain** (mail or web, then download, then infection). Public pcaps cannot show what happens after the first host is infected.

## Result 2: signals in the infection chain

| Signal | Meaning | pcaps | Share |
|---|---|---:|---:|
| Random URI request | HTTP path that looks like random characters | 116 | 40% |
| DGA-like domain | Hard-to-read, random-looking domain names | 83 | 29% |
| UDP burst | One source sends to 20+ external IPs on the same UDP port | 73 | 25% |
| Binary download response | `octet-stream`, `x-msdownload` and similar | 71 | 25% |
| Direct IP access | `Host` header is an IP address | 52 | 18% |
| SWF response | Flash exploit response | 46 | 16% |
| Body is a PE | Response body starts with `MZ` | 40 | 14% |
| Large high-entropy response | Big response that looks compressed or encrypted | 32 | 11% |
| Exploit-kit chain | Payload after an SWF | 28 | 10% |
| Tor gateway name | `onion`-style name lookups | 16 | 6% |
| IP lookup service | Public-IP check service used | 9 | 3% |

One more number as a counter-example: a **"suspicious TLD"** (`.top`, `.xyz`, `.info` and so on) appeared in **202 pcaps (70%)**. A signal this common is almost useless for telling things apart.

Counting the 11 signals above, **at least one** appeared in 185 pcaps (64%) and **none** appeared in 102 (36%). That includes the weakest signals such as random URIs.

## Result 3: each family has its own shape

| Family | n | UDP burst | Download-like | Random URI | DGA-like | Tor |
|---|---:|---:|---:|---:|---:|---:|
| Cerber | 138 | **71** | 42 | 71 | 66 | 0 |
| Locky | 31 | 0 | **22** | 5 | 0 | 0 |
| GlobeImposter | 20 | 0 | 3 | 4 | 7 | **14** |
| Spora | 12 | 0 | 6 | 1 | 0 | 0 |
| Jaff | 10 | 0 | 0 | 1 | 1 | 0 |
| CryptoMix | 7 | 0 | 5 | 7 | 0 | 0 |
| Sage | 7 | 2 | 0 | 0 | 4 | 0 |

- **Cerber**: 71 of 138 show UDP bursts to many external IPs, the most distinctive trait.
- **Locky**: no UDP bursts, but 22 of 31 show download signals.
- **GlobeImposter**: 14 of 20 show Tor-related names.
- **Jaff** (10) shows almost no signal.

## Limitations

- These are **heuristic observation frequencies**. I did not compare against normal browsing captures, so I do not know how often these signals also appear in benign traffic (the false-positive rate). Do not use them as detection criteria.
- It is one year of public captures, mostly exploit-kit and mail-driven chains. Modern ransomware looks different.
- Labels come from file names and may be wrong; 50 pcaps (17%) are unlabeled.
- Thresholds such as "20 or more UDP destinations" are values I chose.
- This is a re-tally of the output table, not a fresh analysis of every pcap from scratch.

## Takeaways

- Public ransomware pcaps are mostly **single-host infection chains**, so they cannot be used to study internal spread.
- Each family has a clear shape (Cerber: UDP, Locky: downloads, GlobeImposter: Tor).
- Common signals (random URIs, suspicious TLDs) mean little on their own.
- Public captures of recent ransomware are rare, so I am recording my own runs in an isolated lab. → [I Ran LockBit 5.0 in an Isolated Lab and the PCAP Was Empty](/en/posts/lockbit5-sandbox/)

## Source

- 2017 posts and attached pcaps on malware-traffic-analysis.net
