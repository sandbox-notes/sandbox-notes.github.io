---
title: "KRACK: How the WPA2 4-Way Handshake Fell Apart"
date: 2026-10-23T05:00:00+09:00
draft: false
series: ["Wi-Fi Security Series"]
tags: ["Wi-Fi", "WPA2", "KRACK", "hostapd", "wpa_supplicant", "AP"]
description: "The first Wi-Fi attack to prove a flaw in the standard itself, re-examined from the point of view of AP development and verification."
---

> Everything here is based on the public paper and public CVEs. Reproduce it only in an isolated lab that you own.

## Introduction

KRACK (Key Reinstallation Attacks), disclosed in October 2017, was the first attack to prove a design flaw in the WPA2 standard itself. It was not a bug in one vendor's product but a gap in the 4-way handshake state machine, so almost every Wi-Fi device of the time was affected.

This series re-examines publicly documented wireless security incidents from the angle of AP development and verification. I picked KRACK first for a simple reason: Dragonblood, FragAttacks and MacStealer all extend the same researcher's work (Mathy Vanhoef), and the lesson "a correct implementation of the standard can still be unsafe" starts here.

## The 4-way handshake in brief

The key question is *when the key is installed*. The client installs the PTK the moment it receives message 3, and at that point its transmit nonce (packet number) and replay counter are reset to 0.

Some terms first. The PMK is the master key that comes from the PSK or from 802.1X authentication. The PTK is a per-session key derived from the PMK, the ANonce, the SNonce and both MAC addresses. The GTK is the group key for broadcast and multicast.

1. AP → STA (message 1): sends the ANonce. The STA derives the PTK from it.
2. STA → AP (message 2): sends the SNonce and a MIC. The AP derives the PTK too and verifies the MIC.
3. AP → STA (message 3): encrypted GTK + MIC + Install flag. **The STA installs the PTK/GTK here.**
4. STA → AP (message 4): acknowledgement. The AP installs the PTK when it receives this.

The trouble is that the wireless link is unreliable. If message 4 is lost, the AP retransmits message 3 as the standard requires, and the STA has to process it again. The standard allowed that retransmission, but it never said the key must not be installed again when it arrives.

## The attack: replaying message 3 to reinstall the key

The attacker blocks message 4 and keeps delivering message 3, so the client installs the same PTK again. Each reinstallation resets the nonce to 0, so data gets encrypted again with nonces that were already used.

To do this the attacker has to sit between the AP and the client. But WPA2 uses the MAC addresses in key derivation, so simply standing up a fake AP breaks the handshake. The paper therefore uses a **channel-based man-in-the-middle**: it runs a clone of the real AP (same MAC and SSID) on another channel, lures the client to it, and relays frames in both directions while selectively blocking or replaying them.

The flow:

1. The attacker takes a channel-based MitM position.
2. Messages 1 to 3 go through normally, and the client installs the PTK and sends message 4.
3. The attacker does not forward message 4 to the AP.
4. The client has already installed the key, so it starts sending encrypted data frames (nonces 1, 2, 3…).
5. The AP times out and retransmits message 3, and the attacker forwards it to the client.
6. The client reinstalls the same PTK and resets the nonce to 0.
7. Later data frames reuse the nonces already used in step 4. Same key + same nonce = same keystream.

The important point is that **the password (PSK) is never exposed**. KRACK does not steal the key. It breaks the premise of the cryptographic primitive: a nonce must be used only once.

## What nonce reuse means

The damage depends on the cipher in use. CCMP allows decryption and replay, while TKIP and GCMP also allow forgery.

| Cipher | Decrypt | Replay | Forge |
|---|---|---|---|
| CCMP (AES) | Yes | Yes | Hard |
| TKIP | Yes | Yes | Yes (MIC key recovery) |
| GCMP | Yes | Yes | Yes (authentication key recovery, both directions) |

Decryption works because of a basic property of stream ciphers. XOR two packets encrypted with the same keystream and you get the XOR of the plaintexts; if you know one plaintext (for example a packet with a fixed header), the other falls out. GCMP is especially dangerous because nonce reuse also reveals the GHASH authentication key, which lets an attacker craft arbitrary packets. Since GCMP is used in 802.11ad/WiGig and later standards, this is not something to wave away.

### The worst case: the all-zero key

Linux wpa_supplicant 2.4/2.5 and Android 6.0 and later, which used it, were hit much harder. After installing a key, the code wiped the TK from memory as a protective measure. When message 3 was retransmitted, it installed that wiped value, that is, **an all-zero key**. In that case the attacker did not need nonce reuse at all and could simply decrypt all traffic. The scene in the paper's demo video where an Android device's traffic is read in full is this case.

A good intention (clearing the key) combined with the state machine to produce the worst possible result, which makes it an especially useful lesson for embedded developers.

## Variants and the CVE list

The same principle applied to every handshake that installs a key, not just the 4-way handshake. Ten CVEs were assigned, and only CVE-2017-13082 is a vulnerability on the **AP side**.

| CVE | Handshake | Key reinstalled | Vulnerable side |
|---|---|---|---|
| CVE-2017-13077 | 4-way | PTK | Client |
| CVE-2017-13078 | 4-way | GTK | Client |
| CVE-2017-13079 | 4-way | IGTK | Client |
| CVE-2017-13080 | Group key | GTK | Client |
| CVE-2017-13081 | Group key | IGTK | Client |
| CVE-2017-13082 | FT (802.11r) reassociation | PTK | **AP** |
| CVE-2017-13084 | PeerKey | STK | Client |
| CVE-2017-13086 | TDLS | TPK | Client |
| CVE-2017-13087 | WNM Sleep Mode | GTK | Client |
| CVE-2017-13088 | WNM Sleep Mode | IGTK | Client |

CVE-2017-13082 is the case where, in 802.11r Fast BSS Transition, an AP that receives a retransmitted Reassociation Request installs the PTK again. No MitM position is needed: merely replaying the Reassociation Request resets the AP-side nonce. Enterprise APs that enable 802.11r for roaming were directly affected.

Reinstalling the GTK (13078, 13080) makes it possible to replay broadcast and multicast frames. On its own that is weaker than decryption, but it can be used to disturb broadcast-based protocols such as ARP and mDNS.

## Patches and mitigations

The root fix is a one-line principle: do not install a key that is already installed. It is simple to implement, but it had to be applied to every handshake path without exception.

On the client side, wpa_supplicant now remembers the TK/GTK/IGTK currently installed and skips installation when the same key arrives again. The all-zero key problem was closed at the same time. Major operating systems shipped patches in October and November 2017, but older Android devices and IoT devices that no longer get updates remain vulnerable today.

On the AP side there are two things:

- **The CVE-2017-13082 fix:** hostapd was changed so that a retransmitted Reassociation Request in an FT reassociation no longer reinstalls the PTK.
- **A mitigation for unpatched clients:** hostapd gained the option `wpa_disable_eapol_key_retries=1`. It turns off retransmission of message 3 and of group key message 1, so the attack cannot work at all. The trade-off is that handshake failures increase in poor radio conditions.

One caution: enabling PMF (802.11w) does **not** stop KRACK. PMF only protects against management frame forgery and has nothing to do with replayed EAPOL key messages.

In 2018 the same research team published a follow-up paper with an improved attack that bypasses some of the patches. In other words, a single patch was not the end of it; the whole state machine had to be re-examined.

## Checkpoints for AP developers

If you build AP products, drop the thought "we are an AP, so client vulnerabilities don't concern us." Today's APs also act as clients through repeaters, mesh backhaul, WDS and similar features.

- [ ] **Check the vendor-forked hostapd:** chip vendor SDKs (such as QSDK) often fork upstream hostapd. Do not look only at the version number. Confirm from source that the October 2017 KRACK patches were actually backported.
- [ ] **Verify CVE-2017-13082 when 802.11r is used:** use the public test scripts (the FT test in krackattacks-scripts) to confirm that the AP does not reinstall the PTK on a retransmitted Reassociation Request.
- [ ] **Audit the wpa_supplicant inside the AP:** mesh backhaul STAs, repeater uplinks and WDS clients use wpa_supplicant. All of the client-side CVEs apply to them.
- [ ] **Decide the `wpa_disable_eapol_key_retries` policy:** at sites with many legacy devices (factories, store POS, IoT), consider turning it on, and measure the handshake failure rate alongside.
- [ ] **Standardize the verification scenarios:** add tests that deliberately lose, retransmit and reorder handshake messages to your regression suite. Normal-operation tests will never catch KRACK.

The biggest lesson is to **suspect state machine re-entry**. For every path that takes the same step twice (retransmission, reconnection, roaming, wake from sleep), ask: is it safe to handle the second time exactly as the first?

## Closing

KRACK showed that a faithful implementation of the standard can still be unsafe. Even the mathematically proven 4-way handshake fell because of one implementation detail: when the key is installed.

The next installment covers **Dragonblood**, the attack on WPA3's SAE. It is the story of WPA3, built to fix WPA2's weaknesses, being broken by side-channel and downgrade attacks within a year of launch.

### References

- [KRACK Attacks official site](https://www.krackattacks.com/): paper, demo videos, FAQ
- Mathy Vanhoef and Frank Piessens, "Key Reinstallation Attacks: Forcing Nonce Reuse in WPA2", ACM CCS 2017
- [krackattacks-scripts (GitHub)](https://github.com/vanhoefm/krackattacks-scripts): client and AP test scripts
- [hostapd/wpa_supplicant security advisory 2017-1](https://w1.fi/security/2017-1/): list of patches
