---
trigger: always_on
description: > **Background research:** See `docs/internal/RESEARCH.md` | **Source of truth:** See `PROJECT.md`
---

# CLAUDE.md — SeekerClaw Project Guide

> **Background research:** See `docs/internal/RESEARCH.md` | **Source of truth:** See `PROJECT.md`

## PROJECT.md — Source of Truth

- Read `PROJECT.md` before any feature work
- After shipping any feature: update **Shipped** section + **Changelog**
- After starting any feature: move it to **In Progress**
- Keep **Limitations** section honest — if it doesn't work, list it
- **One-Liner** and **Elevator Pitch** should always reflect current state
- Update **Stats** periodically (tool count, skill count, lines of code)

## Design Principle: UX First

**Always think about user experience.** This is the top priority when building SeekerClaw. Every UI decision, feature implementation, and config flow should be designed from the user's perspective. Ask: "Is this intuitive? Will the user lose data? Is switching between options seamless?" When in doubt, prioritize ease of use over technical elegance.

## What Is This Project

**SeekerClaw** (package: `com.seekerclaw.app`) is an Android app that turns a Solana Seeker phone into a 24/7 personal AI agent. It embeds a Node.js runtime via `nodejs-mobile` and runs the OpenClaw gateway as a foreground service. Users interact with their agent through Telegram — the app itself is minimal (setup, status, logs, settings).

### Supported Devices

- **Primary:** Solana Seeker (Android 14, Snapdragon 6 Gen 1, 8GB RAM)
- **Secondary:** Any Android 14+ with 4GB+ RAM
- **Note:** OEM-modified ROMs (Xiaomi MIUI, Samsung OneUI) may aggressively kill background services — Seeker's stock Android avoids this.

### Development Phases

- **Phase 1 (PoC):** Mock OpenClaw with a simple Node.js Telegram bot (`grammy`/`telegraf`) that responds to a hardcoded message. Proves Node.js runs on device, Telegram round-trip works.
- **Phase 2 (App Shell):** Replace mock with real OpenClaw gateway bundle. Full setup flow, all screens, watchdog, boot receiver.

## Version Tracking (KEEP UPDATED)

> **When updating OpenClaw or nodejs-mobile, update these version strings in ONE place:**
> **`app/build.gradle.kts`** → `buildConfigField` for `OPENCLAW_VERSION` and `NODEJS_VERSION`
>
> The app version (`versionName` / `versionCode`) is also in `app/build.gradle.kts`.
> All UI screens read versions from `BuildConfig` — no hardcoded strings in Kotlin code.

| Version | Current | Location |
|---------|---------|----------|
| **App** | `1.9.1` (code 18) | `app/build.gradle.kts` → `versionName` / `versionCode` |
| **OpenClaw** | `2026.4.10` | `app/build.gradle.kts` → `OPENCLAW_VERSION` buildConfigField |
| **Node.js** | `18 LTS` | `app/build.gradle.kts` → `NODEJS_VERSION` buildConfigField |

## Tech Stack

- **Language:** Kotlin
- **UI:** Jetpack Compose (Material 3, dark theme only)
- **Theme:** `Theme.SeekerClaw`
- **Min SDK:** 34 (Android 14)
- **Node.js Runtime:** nodejs-mobile (https://github.com/nodejs-mobile/nodejs-mobile) — Node 18 LTS, ARM64
- **QR Scanning:** CameraX + ZXing/ML Kit
- **Encryption:** Android Keystore (AES-256-GCM, `userAuthenticationRequired = false`)
- **Background Service:** Foreground Service with `specialUse` type
- **IPC:** nodejs-mobile JNI bridge + localhost HTTP
- **Database:** SQL.js (WASM-compiled SQLite) — no native bindings needed
- **Build:** Gradle (Kotlin DSL)
- **Distribution:** Solana dApp Store APK (primary), Google Play AAB (secondary), direct APK sideload (fallback)

## Project Structure

```
seekerclaw/
├── app/
│   ├── src/main/
│   │   ├── java/com/seekerclaw/app/
│   │   │   ├── MainActivity.kt              # Single activity, Compose navigation
│   │   │   ├── SeekerClawApplication.kt     # App class
│   │   │   ├── ui/
│   │   │   │   ├── theme/Theme.kt            # Dark theme (Theme.SeekerClaw), Material 3
│   │   │   │   ├── navigation/NavGraph.kt    # Setup → Main (Dashboard/Logs/Settings)
│   │   │   │   ├── setup/SetupScreen.kt      # QR scan + manual entry + notification permission
│   │   │   │   ├── dashboard/DashboardScreen.kt
│   │   │   │   ├── logs/LogsScreen.kt        # Monospace scrollable log viewer
│   │   │   │   └── settings/SettingsScreen.kt
│   │   │   ├── service/
│   │   │   │   ├── OpenClawService.kt        # Foreground Service — starts/manages Node.js
│   │   │   │   ├── NodeBridge.kt             # IPC wrapper for nodejs-mobile
│   │   │   │   └── Watchdog.kt               # Monitors Node.js health, auto-restarts
│   │   │   ├── receiver/
│   │   │   │   └── BootReceiver.kt           # BOOT_COMPLETED → start service
│   │   │   ├── config/
│   │   │   │   ├── ConfigManager.kt          # Read/write config (encrypted + prefs)
│   │   │   │   ├── KeystoreHelper.kt         # Android Keystore encrypt/decrypt
│   │   │   │   └── QrParser.kt               # Parse QR JSON payload
│   │   │   └── util/
│   │   │       ├── LogCollector.kt           # Captures Node.js stdout/stderr
│   │   │       └── ServiceState.kt           # Shared state (StateFlow) for UI
│   │   ├── assets/openclaw/                  # Bundled OpenClaw JS (extracted on first launch)
│   │   ├── res/
│   │   └── AndroidManifest.xml
│   └── build.gradle.kts
├── build.gradle.kts                          # Root build file
├── settings.gradle.kts
├── CLAUDE.md

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sepivip/SeekerClaw](https://github.com/sepivip/SeekerClaw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
