---
trigger: always_on
description: This is **not** a standard single-package Flutter app. The repo root is a workspace with two packages and a build system:
---

# AGENTS.md — CB File Hub

## Project layout

This is **not** a standard single-package Flutter app. The repo root is a workspace with two packages and a build system:

```
cb_file_manager/   ← main Flutter app (all flutter/dart commands run here)
mobile_smb_native/ ← local FFI plugin (SMB/CIFS via libsmb2, path dependency)
justfile           ← primary build orchestrator (uses `just` command runner, requires Git Bash on Windows)
scripts/           ← build, version, and CI helper scripts (bash)
installer/         ← Windows installer configs (Inno Setup, WiX)
```

The `cb_file_manager/windows/llama/` directory holds the bundled llama.cpp
runtime (Vulkan DLLs + `llama-server.exe`, ~69MB). These binaries are **not
committed to git** — they are downloaded at build time (see Local AI runtime
below).

**All `flutter` and `dart` commands must be run from `cb_file_manager/`**, not the repo root.

## Flutter version

Pinned to **3.41.5 stable** (`build.config`, CI workflows). Use this exact version.

## Developer commands

Run from repo root via `just` (requires Git Bash on Windows):

| Task | Command |
|------|---------|
| Show all recipes | `just` |
| Install deps | `just deps` |
| Fetch llama.cpp runtime | `just fetch-llama` |
| Unit/widget tests | `just test` |
| E2E tests (parallel) | `just e2e-parallel` |
| E2E single suite | `just e2e Navigation` |
| E2E single test by name | `just e2e "navigate back to parent with Backspace"` |
| E2E single file | `just e2e-file video_thumbnails_e2e_test` |
| Rerun failed E2E only | `just e2e-failed` |
| E2E plain output (debug) | `just e2e-plain` |
| E2E serial (debug order) | `just e2e-serial` |
| List all E2E test names | `just e2e-list` |
| Analyze | `just analyze` |
| Format | `just format` |
| Format + analyze | `just verify` |
| Clean | `just clean` |
| Deep clean (rebuild) | `just deep-clean` then `just deps` |
| Kill stray E2E processes | `just kill-e2e` |
| Open dashboard | `just dashboard` |

E2E parallel defaults to half the CPU cores (clamped 2..6). Override via:
- `just e2e-parallel "" 4` (positional arg)
- `CB_E2E_MAX_PARALLEL=4 just e2e-parallel` (env var)

Or run Flutter directly from `cb_file_manager/`:

```bash
flutter pub get
flutter test                           # unit/widget tests
flutter test --reporter expanded       # verbose test output
flutter analyze
dart format --output=none --set-exit-if-changed .   # format check (CI uses this)
flutter test integration_test -d windows --dart-define=CB_E2E=true  # E2E
```

## CI pipeline order

CI runs: **format check -> analyze -> unit tests -> E2E (Windows) -> build**. Match this locally with `just verify` before pushing.

## Architecture notes

- **State management:** BLoC (`flutter_bloc`) + Provider + GetIt (service locator in `lib/core/service_locator.dart`)
- **Design system:** Fluent UI on desktop, Material on mobile. Controlled by `DesignSystemConfig` feature flags. The app can run as either `FluentApp` or `MaterialApp`.
- **Localization:** Custom delegate-based (Vietnamese + English) in `lib/config/languages/`. Does **not** use Flutter's `gen-l10n` / ARB files.
- **Navigation:** Tab-based via `TabMainScreen` (`lib/ui/tab_manager/`), not standard Flutter routing.
- **Database:** SQLite via `sqflite` / `sqflite_common_ffi`. On Windows, uses system `winsqlite3.dll` (no bundled DLL).
- **Video:** `media_kit` (primary) + `flutter_vlc_player` (fallback).
- **Streaming:** Built-in HTTP media server via `shelf` (`lib/services/streaming/`).
- **Network browsing:** SMB/CIFS via local `mobile_smb_native` FFI plugin, plus FTP support.
- **Windows native:** Uses `win32` FFI, acrylic backdrop, native tab drag-drop, PiP windowing.

## Feature flags (compile-time)

Passed via `--dart-define=FLAG=value`:

- `CB_E2E=true` — enables E2E test mode (required for integration tests)
- `CB_SHOW_DEV_OVERLAY=true` — shows developer debug overlay
- `CB_ENABLE_FLUENT_DESKTOP_SHELL` — Fluent UI shell toggle

## Key conventions

- **`avoid_print` is disabled** in `analysis_options.yaml` — `print()` is intentionally used in dev/debug code.
- **`main.dart` is 900+ lines** — contains app initialization, window setup, service bootstrap, and the root widget. It is the real wiring diagram of the app.
- **No code generation** — `build_runner` is a dev dependency (for MSIX packaging) but there is no `build.yaml` and no generated Dart code to worry about.
- **Entry point for tests:** unit/widget tests in `cb_file_manager/test/`, E2E in `cb_file_manager/integration_test/`. E2E tooling (parallel runner, Allure adapter, dashboard) lives in `cb_file_manager/tool/`.

## Local AI runtime (llama.cpp)

CB Agent can run local GGUF models on-device via a bundled llama.cpp runtime.

- **Subprocess, not FFI:** the app launches the official `llama-server.exe` as a
  separate OS process and talks to it over HTTP (`lib/services/local_ai/gguf_llama_cpp_runtime.dart`).
  Calling llama.cpp through FFI inside a Dart isolate crashed during GPU tensor
  upload, so the subprocess model is used instead.
- **Binaries are not in git:** the runtime (Vulkan DLLs + `llama-server.exe`,
  ~69MB) lives in `cb_file_manager/windows/llama/`, which is git-ignored. It is

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [coolbirdzik/CB-File-Hub](https://github.com/coolbirdzik/CB-File-Hub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-26 -->
