---
trigger: always_on
description: - Project: Whisper, a Flutter-based LAN collaboration app for nearby devices.
---

# AGENTS.md

## Project Overview

- Project: Whisper, a Flutter-based LAN collaboration app for nearby devices.
- Purpose: send text, files, clipboard content, Android notifications, desktop system audio, and keyboard/mouse input between trusted devices on the same local network.
- Primary users: people moving work between their own computers and phones without a public relay.
- Platforms: Android, macOS, Linux, Windows, and a Next.js product site under `whisper-web/`.
- Important constraints: transfers stay on the LAN, authenticated application data is encrypted end to end (discovery metadata and local storage are not), Linux discovery depends on Avahi, Linux audio depends on PulseAudio or PipeWire Pulse, and Linux keyboard/mouse sharing currently expects X11.

## Setup Commands

- Install Flutter dependencies: `flutter pub get`
- Regenerate Drift database code after schema changes: `dart run build_runner build --delete-conflicting-outputs`
- Regenerate localizations after ARB changes: `flutter gen-l10n`
- Run the Flutter app on the current platform: `flutter run`
- Run the signed macOS debug app used by this repo: `./script/build_and_run.sh`
- Run macOS debug under LLDB: `./script/build_and_run.sh --debug`
- Stream macOS app logs/telemetry: `./script/build_and_run.sh --logs`
- Package a macOS DMG: `./script/build_and_run.sh package-macos`
- Install product-site dependencies: `cd whisper-web && npm install`
- Start the product site: `cd whisper-web && npm run dev`
- Build the product site: `cd whisper-web && npm run build`

## Verification Commands

- Analyze Dart/Flutter code: `flutter analyze`
- Run all Flutter tests: `flutter test`
- Run one focused test: `flutter test test/<name>_test.dart`
- Run the remote-input key matrix suite: `./script/test_remote_input_keys.sh`
- Verify the macOS app builds, signs, launches, and has a running process: `./script/build_and_run.sh --verify`
- Check product-site linting when working in `whisper-web/`: `cd whisper-web && npm run lint`

CI currently runs `flutter pub get`, `flutter analyze`, and `flutter test` on pull requests and pushes to `main`.

## Project Structure

- `lib/main.dart`: Flutter entry point, locale/theme bootstrapping, desktop window initialization.
- `lib/page/`: main screens such as device discovery, conversation, app list, and settings.
- `lib/widget/`: reusable UI pieces for chat, composer, banners, dialogs, and desktop workspace layout.
- `lib/state/`: app state coordinators for connections, auto-connect, sessions, transfers, shutdown, and discovery throttling.
- `lib/socket/`: WebSocket server/client orchestration, auth flow, message dispatch, file transfer, profile refresh, audio, and remote input routing. File path validation, portable names, and verified publication are separated into `file_path_policy.dart`, `transfer_file_name.dart`, and `verified_file_publisher.dart`.
- `lib/model/`: Drift database, device/message/transfer models, and generated database code.
- `lib/audio/`: audio sharing protocol, codecs, capture/playback abstractions, transport, and runtime coordinator.
- `lib/remote_input/`: keyboard/mouse sharing protocol, topology/layout, native platform bridges, transport, and coordinator.
- `lib/helper/`: platform helpers for files, settings, notifications, background services, desktop startup, and general utilities.
- `lib/l10n/`: ARB files and generated Flutter localization classes for Chinese, English, and Spanish.
- `lib/theme/`: Material 3 theme and `WhisperPalette` tokens.
- `android/`, `ios/`, `macos/`, `linux/`, `windows/`: native runners and platform plugins.
- `test/`: Flutter unit/widget/source tests; many tests verify source-level platform integration and protocol behavior.
- `script/`: local build, run, signing, packaging, and focused test helpers.
- `.github/workflows/`: CI and multi-platform release packaging.
- `docs/superpowers/`: existing design specs and implementation plans.
- `whisper-web/`: Next.js 15 product introduction site.

## Generated And Large Files

- Do not hand-edit `lib/model/LocalDatabase.g.dart`; update the Drift source and run build runner.
- Do not hand-edit `lib/l10n/app_localizations*.dart`; update the ARB files and run `flutter gen-l10n`.
- Treat Flutter generated plugin registrants under platform folders as generated unless the platform tooling requires a manual change.
- Avoid touching `build/`, `.dart_tool/`, `.flutter-plugins-dependencies`, `dist/`, and other local build outputs.
- `pubspec.lock` is tracked for this app; update it only as part of dependency changes.

## Code Style

- Follow the existing Flutter/Dart style and `flutter_lints` baseline in `analysis_options.yaml`.
- Keep platform-specific behavior behind the existing helper, coordinator, or native plugin boundaries.
- Prefer explicit protocol/state transitions over implicit side effects, especially in `lib/socket/`, `lib/audio/`, and `lib/remote_input/`.
- Keep UI strings localized through ARB files; avoid adding hardcoded user-facing copy in widgets.
- Use existing singleton coordinators and managers when extending current flows; introduce new abstractions only when they reduce real duplication.
- Preserve current Material 3 theme tokens from `AppTheme` and `WhisperPalette` when adding UI.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lawnvi/whisper](https://github.com/lawnvi/whisper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
