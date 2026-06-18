---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Soduto (Android)** — a downstream fork of [KDE Connect Android](https://invent.kde.org/network/kdeconnect-android) maintained by Sannidhya Roy. Soduto is a KDE Connect-compatible cross-device communication app; the Android fork exists to ship bug fixes, reliability improvements, and new plugins faster than upstream, while staying close enough to upstream to keep rebasing tractable.

**Strategic direction:** minimal divergence. Every addition is designed so it can be stripped cleanly when upstream eventually ships an equivalent. Do not propose deep architectural changes. Always ask: will this be easy to remove when upstream catches up?

### Identity

| Key | Value |
|-----|-------|
| App name | Soduto |
| Application ID | `com.thenoton.soduto` |
| Namespace (R/BuildConfig) | `org.kde.kdeconnect_tp` (unchanged from upstream) |
| Upstream repository | <https://invent.kde.org/network/kdeconnect-android.git> |
| Upstream branch | `upstream/master` |
| Downstream repository | <https://github.com/sannidhyaroy/soduto-android.git> |
| Downstream branch | `main` |

## Versioning

Soduto Android uses **independent versioning**, decoupled from the upstream KDE Connect version number.

| Field | Value / Scheme |
|-------|----------------|
| `versionName` | Soduto's own semver: `MAJOR.MINOR.PATCH` (e.g. `4.0.0`) |
| `versionCode` | `MAJOR × 10000 + MINOR × 100 + PATCH` (e.g. `4.0.0 = 40000`) |
| `BuildConfig.KDE_VERSION` | Upstream KDE Connect base (e.g. `"1.35.5"`) |

Both values are defined as named variables at the top of `build.gradle.kts` (`sodutoVersion`, `sodutoVersionCode`, `kdeVersion`) so upstream versionCode/versionName bumps no longer conflict with our lines.

**Version bump rules:**
- Patch (`4.0.1`) — bug fix downstream releases
- Minor (`4.1.0`) — new downstream features
- Major (`5.0.0`) — significant milestone, coordinated with macOS Soduto at major level

The upstream KDE Connect base lives in `kdeVersion` in `build.gradle.kts` and `BuildConfig.KDE_VERSION` at runtime (displayed in the About screen). Update `kdeVersion` when rebasing onto a new upstream release.

## Build Commands

```bash
# Build debug APK
./gradlew assembleDebug

# Run lint
./gradlew lintDebug

# Run all unit tests
./gradlew testDebug

# Run a single test class
./gradlew testDebug --tests "org.kde.kdeconnect.ClassName"

# Generate third-party license report
./gradlew generateLicenseReport
```

**Build config:** minSdk 23, targetSdk 35, compileSdk 36, Kotlin JVM target Java 11, core library desugaring enabled.

## Architecture

### Core Layers

**Application singleton (`KdeConnect.kt`)** — manages all `Device` instances, discovery callbacks, and global shared preferences.

**`BackgroundService.kt`** (foreground service) — owns `LinkProvider` instances, monitors network connectivity, manages the app lifecycle when not in the foreground.

**`Device.kt`** — central class representing a paired remote device. Holds active `BaseLink`s (transport connections), loaded `Plugin`s, and the `PairingHandler`. Dispatches incoming `NetworkPacket`s to the appropriate plugin.

**`NetworkPacket`** — JSON envelope for all inter-device communication. Every packet has a `type` string that routes it to the matching plugin.

### Transport Backends (`backends/`)

- `lan/` — WiFi/LAN via TCP + mDNS/NSD discovery
- `bluetooth/` — Bluetooth RFCOMM with a `ConnectionMultiplexer` for multiplexing virtual channels
- `loopback/` — in-process loopback for local testing

### Plugin System (`plugins/`)

Each feature is a `Plugin` subclass annotated with `@PluginFactory`. KSP (`ClassIndexKSP`) generates a registry at compile time so plugins are discovered without reflection at runtime. Plugins receive `NetworkPacket`s via `onPacketReceived()` and send packets via `device.sendPacket()`. Plugin preferences are split into device-specific and global.

**Upstream plugins:** `battery`, `clipboard`, `connectivityreport`, `contacts`, `digitizer`, `findmyphone`, `findremotedevice`, `inputdevicesreceiver`, `mousepad`, `mousereceiver`, `mpris`, `mprisreceiver`, `notifications`, `ping`, `presenter`, `receivenotifications`, `remotekeyboard`, `runcommand`, `sftp`, `share`, `sms`, `systemvolume`, `telephony`

**Soduto-added plugins:**
- `lock/` — `LockPlugin.kt` reads and controls the lock state of the remote device; sends `requestLocked` on connect, exposes `remoteIsLocked`, provides Lock/Unlock menu actions
- `lockreceiver/` — `LockReceiverPlugin.kt` handles remote lock commands on Android; reports lock state changes via screen broadcasts; executes `lockNow()` via `DevicePolicyManager` when Device Admin is active (requires `KdeConnectDeviceAdminReceiver` registration in the manifest)
- `systemvolumeprovider/` — `SystemVolumeProviderPlugin.kt` exposes Android audio streams to desktop via the `kdeconnect.systemvolume` packet type; `AndroidSinksProvider.kt` discovers streams and tracks volume changes via `ContentObserver`; settings UI in `SystemVolumeProviderSettingsFragment.kt`

### Security


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sannidhyaroy/soduto-android](https://github.com/sannidhyaroy/soduto-android) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
