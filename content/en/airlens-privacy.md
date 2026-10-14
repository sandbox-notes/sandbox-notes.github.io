---
title: "Airlens Privacy Policy"
date: 2026-10-14
description: "What the Airlens Android app reads, where it is kept, and the one connection it makes."
---

**Effective date:** 14 October 2026
**Published by:** Sandbox Notes
**Contact:** sandbox.notes.lab@gmail.com

## In short

Airlens has no server. There is no account to create, no analytics or
advertising code in the app, and no mechanism by which anything it measures
could reach the developer — because there is nowhere for it to be sent.
Everything Airlens measures stays on your phone until you decide to export or
share it.

There is exactly one outbound connection in the entire app, and it only happens
when you press a button. It is described in full below.

## What Airlens does not do

- No account, sign-in, or user identifier of any kind
- No analytics, telemetry, crash reporting, or advertising SDK
- No data transmitted to the developer
- No background collection — measurement runs only while the app is open
- **No location fix is ever requested.** Airlens does not read GPS, does not
  use fused location, and does not derive, store, or transmit a position

## Why Airlens asks for location permission

This is the question the permission prompt cannot answer, so it is answered
here.

Android gates all Wi-Fi scanning behind location permission. `startScan()` and
`getScanResults()` — the two calls that list nearby access points — return
nothing without `ACCESS_FINE_LOCATION`, on every Android version Airlens
supports, including Android 13 and later. The operating system refuses the call
with "has no location permission" and hands back an empty list.

The permission is therefore a precondition imposed by Android for reading radio
information, not something Airlens needs in order to know where you are. It
does not request a location, and holding the permission does not cause one to
be collected.

## Permissions and what each is for

| Permission | Used for |
|---|---|
| `ACCESS_FINE_LOCATION`, `ACCESS_COARSE_LOCATION` | Required by Android before it will return Wi-Fi scan results. No position is requested or stored. |
| `ACCESS_WIFI_STATE` | Reading scan results and the state of the connected link |
| `CHANGE_WIFI_STATE` | Asking the system to start a Wi-Fi scan |
| `ACCESS_NETWORK_STATE` | Reading the current network's configuration — proxy, DNS, connection state |
| `INTERNET` | The speed test only. See below. |
| `NEARBY_WIFI_DEVICES` | Declared for Android 13+ compatibility. Airlens never requests it at runtime, so you will not be asked for it. |

Airlens does not request camera, microphone, contacts, storage, phone, SMS, or
background location permissions.

## What Airlens reads, and where it is kept

While you use it, Airlens reads:

- **Nearby access points** — network name (SSID), hardware address (BSSID),
  signal strength, channel, and the raw beacon information elements the access
  point broadcasts
- **The connected link** — network name, hardware address, signal, negotiated
  rate, frequency, and the transitions between access points as they happen
- **Devices on your local network**, only when you run a LAN scan — IP address,
  hardware address, hostname where one is published, and which of a small set
  of common ports respond
- **This device's security posture** — certificate authorities installed in the
  user trust store, whether a screen lock is set, whether USB or wireless
  debugging is enabled, and the current network's proxy and DNS configuration
- **Walk tests and roaming events you record** — signal samples once per second
  for the duration of a recording, and the access point transitions during it

All of it is written to storage private to the app on your own device. It is
removed when you uninstall Airlens or clear its data. None of it is uploaded.

Reports you export are ordinary files written to `Downloads/Airlens`. They
belong to you, they are not visible to the developer, and they remain on the
device after Airlens is uninstalled until you delete them.

## The one outbound connection

The **speed test** is the only part of Airlens that contacts the internet, and
only when you press *Run speed test*.

It transfers randomly generated data to and from `https://speed.cloudflare.com`,
a service operated by Cloudflare, Inc., and measures how fast that transfer
goes. As with any internet request, this makes your IP address visible to that
service. No identifier, no network name, and no measurement from elsewhere in
the app is attached to it, and Airlens sends nothing about you.

Cloudflare's handling of that request is governed by its own privacy policy:
<https://www.cloudflare.com/privacypolicy/>

If you never run a speed test, Airlens makes no outbound connection at all.

The LAN scan contacts devices **on your own local network only**. Those packets
do not leave your network.

## Reports you choose to share

An Airlens report can contain network names, hardware addresses, and an
inventory of devices on the scanned network. Exporting and sharing are always
started by you.

Once you send a report to someone, its contents are subject to whatever service
you sent it through and to whoever receives it. If you are producing a report
for a client, it is worth checking whether the LAN device inventory belongs in
it before sending.

## Purchases

Airlens Pro is sold either as a 7-day pass or as a permanent unlock. Both are
one-off purchases processed by Google Play. Neither is a subscription and
neither renews by itself.

Payment is handled entirely by Google; Airlens never sees or stores card
details or billing information. What has been bought is read back from Google
Play rather than kept as a record this app maintains, which is why a purchase
survives reinstalling the app or moving to another phone on the same Google
account. Google's handling of the transaction is covered by the Google Play
Terms of Service and Google's privacy policy.

## Children

Airlens is a network diagnostic tool, is not directed at children, and collects
no personal information from anyone regardless of age.

## Changes to this policy

If Airlens ever gains a feature that changes any of the above, this policy will
be updated before that feature ships, and the effective date at the top will
change. Material changes will also be noted in the app's release notes.

## Contact

sandbox.notes.lab@gmail.com
