---
trigger: always_on
description: > **Read this file first.** It is the source of truth for project intent, architecture, status, and constraints.
---

# AGENTS.md — Context for AI agents working on this repo

> **Read this file first.** It is the source of truth for project intent, architecture, status, and constraints.

## What this repository is

**Lumina eUICC** is a Flutter-first removable / programmable eUICC manager with
a modern Material 3 interface. It uses the vendored OpenEUICC/lpac LPA stack
behind a thin native bridge.

| Field | Value |
|---|---|
| GitHub | https://github.com/Syngnat/lumina-euicc (public) |
| Owner | Syngnat |
| App ID | `top.syngnat.lumina.euicc` |
| Display name | Lumina eUICC |
| Primary platforms | Android 9+ (API 28) and iOS 13+; Flutter UI |
| Not a goal | Managing **internal** phone eSIM without system privilege |

Non-root operation is a hard product constraint: never add Root/Magisk/Shizuku flows, system-app installation requirements, private iOS entitlements, or privileged telephony permissions. Android's supported in-phone path is OMAPI to a removable eUICC whose ARA-M authorizes at least one current Lumina APK signer; USB CCID is the alternative. iOS supports only an external USB CCID reader through CryptoTokenKit—ordinary apps cannot send APDUs to the iPhone SIM slot. Release APKs from `0.1.1` onward use one stable four-current-signer set (Lumina, Sakura, ShiinaSekiu Community, and 9eSIM). The three community identities are reproducible from pinned public source; the Lumina identity remains private and anchors Android update security.

### Product goal (user request)

- Existing removable-eUICC managers are useful but visually dated
- User wants **complete removable-card capability** with a **better UI**
- Prefer **not writing app logic in Java**; accepted architecture is **Flutter UI + thin Kotlin/Swift bridges + OpenEUICC/lpac LPA stack**
- User develops on **Windows**; this repo was scaffolded on a low-RAM VPS that **cannot** reliably build full Flutter/Android APKs

## What it is NOT

- Not a pure-Flutter eUICC stack (OMAPI/CryptoTokenKit/APDU access needs native platform code)
- Not a full privileged OpenEUICC system app for internal eSIM
- Not a web app
- Not finished end-to-end: one exact card/device combination has read-only ARA-M/profile-list evidence, but mutations, named-model coverage, and USB CCID still require real-device validation

## Architecture

```text
Flutter (Dart) — Material 3 UI
    MethodChannel: top.syngnat.lumina.euicc/bridge
    EventChannel:  top.syngnat.lumina.euicc/task_events
        ↓
Platform bridge
  Android: EuiccBridgePlugin.kt → OpenEUICC app-common + lpac-jni
  iOS:     EuiccBridgePlugin.swift → LpacSession + vendored lpac C
        ↓
Android OMAPI / USB CCID, or iOS CryptoTokenKit / USB CCID
```

### Key paths

| Path | Role |
|---|---|
| `lib/` | Flutter UI, models, providers, MethodChannel client |
| `lib/services/euicc_bridge.dart` | Dart API surface (what Flutter pages call) |
| `lib/pages/` | Home, download/QR, compatibility, settings |
| `android/app/.../EuiccBridgePlugin.kt` | Real LPA + read-only diagnostics; mock fallback is debug-only |
| `android/app/.../LuminaApplication.kt` | Hosts OpenEUICC `DefaultAppContainer` |
| `ios/Runner/EuiccBridgePlugin.swift` | iOS Method/EventChannel bridge and platform capability boundary |
| `ios/Runner/LpacSession.swift` | Swift facade over the vendored lpac C API |
| `ios/Runner/SmartCardTransport.swift` | CryptoTokenKit USB CCID/APDU transport; no iPhone SIM-slot access |
| `ios/Runner/Native/` | Objective-C-compatible C facade plus per-source lpac/cJSON compilation units |
| `third_party/OpenEUICC/` | Vendored upstream sources; preserve each component's license files |
| `docs/FEATURE_PARITY.md` | Native/API capability mapping; not proof of Flutter UI exposure |
| `docs/NATIVE_INTEGRATION.md` | Integration status & build notes |
| `docs/COMMUNITY_SIGNING.md` | Four-signer identity, ARA-M, migration, provenance, and update policy |
| `docs/SUPPORTED_CARDS.md` | Evidence-based card candidate matrix and USB limitations; not a hardware certification |
| `LICENSE` / `LICENSES_SCOPE.md` | GPL-3.0-only text and precise project/third-party license boundary |
| `NOTICE.md` / `THIRD_PARTY_NOTICES.md` | Copyright attribution and component-specific third-party terms |

## Unprivileged removable-eUICC capability

| Capability | Status in code |
|---|---|
| List channels (OMAPI / USB) | Android real path and iOS CryptoTokenKit USB path are implemented in source; Android debug-only mock and release unavailable states are explicit; one exact Android OMAPI combination opened ISD-R, while every USB path remains unvalidated |
| List profiles | UI/bridge path implemented; one exact, model-unknown 9eSIM-card/device combination listed real profiles with `0.1.1` |
| Enable / disable profile | UI/bridge path implemented; hardware validation pending |
| Delete / rename profile | UI/bridge path implemented; hardware validation pending |
| Keep-alive reminders | Android uses AlarmClock/AlarmManager; iOS uses UserNotifications. Neither path persists plaintext ICCID or writes reminder data to the eUICC |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Syngnat/lumina-euicc](https://github.com/Syngnat/lumina-euicc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-02 -->
