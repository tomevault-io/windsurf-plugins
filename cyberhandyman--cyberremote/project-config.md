---
trigger: always_on
description: A free, open-source, ad-free Android app that controls Apple TV (tvOS 13+,
---

# CLAUDE.md — Open-Source Android Remote for Apple TV (Companion Protocol)

## What this project is

A free, open-source, ad-free Android app that controls Apple TV (tvOS 13+,
focus on tvOS 15–26) over the local network, by natively implementing Apple's
**Companion Link protocol** — the same protocol family used by the iOS Control
Center remote widget. No cloud, no bridge server, no companion daemon on a
Mac/PC. The phone talks directly to the Apple TV.

Core v1 features (parity with the iOS remote widget for navigation + typing):
- D-pad / select / menu(back) / home, play-pause, volume, power (wake/sleep)
- Touchpad swipe navigation
- **Full keyboard text input** — when a text field is focused on the Apple TV,
  the phone's soft keyboard types directly into it (search boxes, passwords,
  URLs), exactly like the iOS remote's keyboard. This is a headline feature,
  not an afterthought.
- App list + launch

- Repo layout goal: publishable on GitHub, installable via GitHub Releases APK
  (and later F-Droid).
- License: MIT (protocol knowledge comes from community reverse-engineering
  projects, primarily pyatv, which is MIT).
- Privacy: the app makes **zero** network connections other than the local
  LAN connection to the Apple TV. No analytics, no ads, no telemetry.
- Naming/trademark: do NOT name the app "Apple TV Remote". Use a neutral name
  (working name: **Companion Remote**). "Apple TV" may only appear
  descriptively ("works with Apple TV") with a disclaimer that this is not an
  Apple product.

## Non-goals (v1)

- No "now playing" metadata / artwork (that requires the MRP-over-AirPlay-2
  tunnel — an order of magnitude more work; may become v2).
- No AirPlay streaming, no screen mirroring.
- No iOS version, no IR, no Bluetooth HID fallback.
- No support for Apple TV 3 and older (DMAP/DACP legacy protocol).

## Architecture

Two Gradle modules, strict separation:

```
companion-remote/
├── CLAUDE.md
├── protocol/          # Pure Kotlin/JVM library. ZERO Android dependencies.
│   └── src/main/kotlin/.../
│       ├── opack/     # OPACK encoder/decoder
│       ├── tlv8/      # TLV8 encoder/decoder (HAP pairing data)
│       ├── crypto/    # SRP-6a (HAP variant), HKDF, Ed25519, X25519,
│       │              # ChaCha20-Poly1305 session (BouncyCastle)
│       ├── hap/       # Pair-setup (M1–M6), pair-verify (M1–M4)
│       ├── companion/ # Frame codec, encrypted session, HID commands,
│       │              # touch gestures, RTI text input, app list/launch,
│       │              # power, events
│       └── client/    # High-level CompanionClient facade (coroutines)
├── cli/               # Small JVM command-line harness that uses :protocol
│                      # (scan/pair/command). THE primary integration-test
│                      # tool — protocol bugs are debugged here, not in the app.
├── app/               # Android app. Jetpack Compose + Material 3.
│                      # Discovery (NsdManager), pairing UI, remote UI,
│                      # credential storage. Depends only on :protocol.
└── references/        # git submodules / clones, read-only, NOT shipped:
    ├── pyatv/                    # canonical reference implementation
    └── node-appletv-remote/     # TypeScript port, good for porting logic
```

Rules:
- `:protocol` must compile and pass all tests with plain `java`/JUnit on any
  machine — this is what makes the protocol fully unit-testable without a
  device and reusable by other projects.
- All I/O in `:protocol` goes through a tiny `Transport` interface
  (send/receive bytes) so tests can drive it with recorded byte streams.
- `:app` contains no protocol logic. If you find yourself writing byte
  handling in `:app`, stop and move it to `:protocol`.

## Tech stack

- Kotlin 2.x, coroutines. Gradle Kotlin DSL, version catalog.
- Android: minSdk 26, targetSdk latest stable. Jetpack Compose, Material 3.
- Crypto: **BouncyCastle** (works identically on JVM tests and Android).
  Do not use Android-only providers in `:protocol`.
- mDNS: `NsdManager` on Android (acquire a `WifiManager.MulticastLock`
  first — discovery silently fails without it). In `:cli` use jmDNS.
- Persistence: Jetpack DataStore for credentials; wrap secrets with an
  Android Keystore–backed key. (Do not use the deprecated
  EncryptedSharedPreferences.)
- CI: GitHub Actions — JVM tests on every push, assembleRelease on tag,
  attach APK to GitHub Release.

## Protocol summary (verified against pyatv docs)

Canonical spec: https://pyatv.dev/documentation/protocols/ (section
"Companion Link"). Source of truth for behavior: `references/pyatv/pyatv/`
(`protocols/companion/`, `auth/`, `support/opack.py`). When this file and
pyatv disagree, **pyatv wins** — port, don't invent.

### Discovery
- Zeroconf service type: `_companion-link._tcp.local.`
- TXT records include `rpMd` (model, e.g. `AppleTV6,2`), `rpVr` (version),
  `rpFl` (flags). Most other values rotate for privacy; ignore them.
- The port can change after reboot; re-resolve on every connect, never cache.

### Frame format (TCP)
```
| Frame type (1 byte) | Length (3 bytes, big-endian) | Payload |
```
Frame types used: `0x03` PS_Start, `0x04` PS_Next, `0x05` PV_Start,
`0x06` PV_Next, `0x08` E_OPACK (all remote-control traffic after auth).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cyberhandyman/CyberRemote](https://github.com/cyberhandyman/CyberRemote) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-31 -->
