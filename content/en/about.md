---
title: "About"
date: 2026-10-04
description: "What this blog covers and the principles behind it."
---

## What this is

A place where I run malware in an isolated lab and write down **what I verified and what I could not**. It is also a public log of my path toward becoming a security professional.

## Topics

- **Lab setup**: VMware isolated networking, REMnux, FlareVM, INetSim, and the places I got stuck
- **Sample analysis**: packet captures, local behavior, and cross-checks against public reports
- **Lessons learned**: symptoms, causes and fixes in tables
- **Detection and defense**: turning findings into IOCs and detection rules

## Writing principles

1. **Observation versus assumption.** What I saw myself is kept apart from what I took from public sources.
2. **No samples.** Only hashes and links to public analyses.
3. **Defanged IOCs.** Domains and addresses are written with `[.]` and similar.
4. **No personal identifiers.** Hostnames, usernames and paths are removed before publishing.
5. **Limits are stated.** A different environment may give different results, and I say so.

## Environment

- Hypervisor: VMware Workstation
- Detonation: FlareVM (Windows)
- Fake internet and capture: REMnux (INetSim, dnsmasq, tcpdump)
- Network: a dedicated virtual network cut off from the host and the internet, with a snapshot before every run

## Author

I run Sandbox Notes. I am studying malware analysis with the goal of working in security, and I write up what I try in my own lab.

I aim to leave the sticking points and how I got past them in the posts instead of hiding them.

## Disclaimer

Everything here is for education and research. These are my own views, not those of any organization. Do not use the techniques described against systems you do not have permission to test. If you spot a mistake, tell me and I will check and correct it.

## Contact

sandbox.notes.lab [at] gmail.com (please replace `[at]` with `@`; this keeps spam bots away)
