---
trigger: always_on
description: Compact reference for OpenCode sessions working on this repo. Every line answers "would an agent miss this without help?"
---

# AGENTS.md

Compact reference for OpenCode sessions working on this repo. Every line answers "would an agent miss this without help?"

## Project Snapshot

- App: `WanSync` / `onelap_strava_sync` — Flutter + Dart
- Purpose: sync OneLap FIT activity files to Strava and Xingzhe
- Entry: `lib/main.dart`
- Platform support: Android (primary), iOS, macOS, Linux, Windows, Web

## Source Layout

```
lib/
  main.dart              — app bootstrap, MaterialApp, share-intake wiring
  models/                — immutable data classes (OneLapActivity, SyncRecord, etc.)
  services/              — all business logic (network, sync, dedupe, settings, state)
  screens/               — UI screens (home, settings, share-confirm, OAuth, history)
test/
  models/                — model tests
  screens/               — screen tests
  services/              — service tests
```

Platform folders (`android/`, `ios/`, etc.) — only edit when the task requires platform-specific changes.

## Key Technical Facts

- **Two destination platforms**: Strava (upload + poll via REST API or web session) and Xingzhe (upload + poll via web session). Both can be enabled/disabled independently.
- **Strava upload modes**: API mode uses OAuth (`strava_client.dart`), web mode uses session cookies (`strava_web_client.dart`). Switchable in settings.
- **Deduplication**: two-layer — `fingerprint` (SHA256 of FIT bytes + recordKey + startTime) and `dedupeKey` (startTime + distance) as a stable fallback. See `dedupe_service.dart` and `state_store.dart`.
- **Coordinate conversion**: optional GCJ-02 → WGS84 rewrite before upload, using `fit_tool` package.
- **Strava OAuth**: done via `webview_flutter` in `strava_auth_screen.dart`. Tokens flow back into `SettingsService`.
- **OneLap login**: username/password-based, session via `dio_cookie_manager`.
- **State persistence**: `state.json` in app documents directory — contains synced fingerprints, dedupe keys, sync history (last 500 records), sync result banners (last 7).
- **Settings persistence**: all credentials via `flutter_secure_storage`, preferences via `shared_preferences`.
- **HTTP client**: `Dio` everywhere — with explicit timeouts (30s connect/receive) and cookie manager for OneLap.
- **Error types**:
  - `StravaRetriableError` / `StravaPermanentError` — for API 4xx vs 5xx distinction
  - `StravaWebSessionExpiredError` / `StravaWebUploadError` — web upload session expiry and upload failures
  - `OnelapRiskControlError` — risk-control triggered, sync aborted gracefully
  - `_isIdempotentSuccess()` in `sync_engine.dart` — catches "already uploaded" / duplicate responses as success

## Setup

```bash
flutter pub get
```

SDK: `^3.11.3` (pubspec.yaml).  
CI runs Flutter `3.41.5` stable (see `.github/workflows/ci.yml`).

## Build

```bash
# Release APK (primary target)
flutter build apk --release --dart-define=FLUTTER_IMPELLER_ENABLED=false

# Debug (signed with ~/.android/debug.keystore)
flutter build apk --debug
```

Debug builds use the default debug keystore at `~/.android/debug.keystore`. If it doesn't exist, create it with:
```bash
keytool -genkey -v -keystore ~/.android/debug.keystore -storepass android -alias androiddebugkey -keypass android -keyalg RSA -keysize 2048 -validity 10000 -dname "CN=Android Debug,O=Android,C=US"
```

## Verification Flow

```bash
dart format --output=none --set-exit-if-changed lib test
flutter analyze
flutter test
```

Run in this order. CI enforces the same pipeline.

- **MUST pass all three before merging to main**: `dart format`, `flutter analyze`, `flutter test`. Don't rely on CI alone — run locally first.

### Targeted test commands

```bash
flutter test test/services/strava_client_test.dart          # single file
flutter test --plain-name "exact test name"                  # single test
flutter test -r expanded                                     # verbose output
```

When changing parsing, dedupe, sync, or persistence, add or update a test — coverage for these areas already exists in `test/services/`.

## Code Style

Follow existing repository patterns first. Standard Dart/Flutter conventions apply where the repo is silent.

- **Imports**: relative for local files (`../services/`), package imports for Flutter/external (`package:flutter/material.dart`). No mixed styles in the same file.
- **Naming**: `snake_case.dart` files; storage keys `UPPER_CASE` (`ONELAP_USERNAME` etc.).
- **Format**: `dart format` only — no hand-formatting. Respect trailing commas and multiline wrapping from the formatter.
- **Types**: prefer explicit types for fields and return values (matching existing code). Use nullable types only when `null` is a real state. Use `required` named parameters for mandatory inputs. Prefer `const` constructors and values.
- **i18n**: UI copy is Chinese, README is bilingual Chinese/English. Preserve both.
- **Theme**: Material 3, `Colors.deepOrange` seed.
- **UI pattern**: check `mounted` before using `context` after `await` in stateful widgets.
- **State/persistence**: keep persisted key names stable once released. `state.json` has backward-compat structure (`synced` → `platforms` per fingerprint, `dedupeKeys` → `fingerprint` + `platforms`). Prefer additive migrations over destructive resets.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Tyan66666/WanSync](https://github.com/Tyan66666/WanSync) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
