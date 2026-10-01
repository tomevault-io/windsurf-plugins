---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Harness Desktop — a Flutter app that lists Harness machines and attaches xterm terminals to the
agents running on them. **macOS and Linux (Ubuntu) are both real, released targets** — first-run
provisioning (`lib/bootstrap/environment_provisioner.dart`), self-update
(`lib/update/desktop_updater.dart`), and packaging (`scripts/upload-desktop.sh` /
`scripts/upload-desktop-linux.sh`, see RELEASE.md) all branch per-OS internally rather than being
separate code paths. The Windows runner exists but is unexercised. Package name is `harness`
(`import 'package:harness/...'`). It lives under `desktop/` in the `autonomous-harness` repo, beside
the CLI (`../cli`) and the backend (`../backend`); a few comments still point at `autonomous-code`,
the older backend checkout, which is a sibling of this repo rather than part of it.

**Grid was removed on 2026-09-11, when the project was killed**: no `lib/grid/`, no `lib/share/`, no
Settings ▸ Providers or Share Intelligence, no provider pill, model picker, node dashboard or
usage-limit card, and first-run setup no longer installs the Grid CLI. The last commit that still
has all of it is the `archive/grid` branch — bring anything back from there rather than rewriting
it from memory. The harness CLI (`autonomous-harness`) still carries its own grid code
(`gridLaunch.ts`, `gridWebMcp.ts`, `agent_retarget`, `harness grid login`); nothing here calls it.

## Toolchain and commands

`pubspec.yaml` pins `sdk: ^3.13.0`, i.e. **Flutter ≥ 3.47 / Dart ≥ 3.13**. An older Flutter fails at
`flutter pub get` ("version solving failed") and every command below fails with it — check
`flutter --version` first.

The macOS project is migrated to **Swift Package Manager** (`macos/Runner.xcodeproj` references
`FlutterGeneratedPluginSwiftPackage`). Run `flutter config --enable-swift-package-manager` once, then
`flutter pub get` — the generated `macos/Flutter/ephemeral/Packages/FlutterGeneratedPluginSwiftPackage/Package.swift`
only lists the plugin dependencies when SPM is on at `pub get` time. With SPM off, `flutter run` falls
back to CocoaPods and rewrites tracked files (`project.pbxproj`, `contents.xcworkspacedata`, the
`Flutter-*.xcconfig`s) and adds `macos/Podfile`; revert those rather than committing them.

```bash
flutter pub get
flutter analyze                                   # lints: package:flutter_lints, no custom rules
flutter test                                      # whole unit/widget suite (test/)
flutter test test/terminal_session_test.dart      # one file
flutter test test/ws_conn_test.dart --plain-name "reconnects"   # one test by name substring
flutter run -d macos                              # or: flutter run -d linux
bash scripts/build-macos-debug.sh                  # pins the host's release renderer
flutter build macos --release
flutter build linux --release                     # Ubuntu build host only — no cross-compiling
```

Native integration fixtures need a device and the test-mode environment:

```bash
FLUTTER_TEST=1 flutter test -d macos --no-pub integration_test/native_terminal_e2e_test.dart
FLUTTER_TEST=1 flutter test -d macos --no-pub integration_test/native_workspace_e2e_test.dart
```

Both use in-memory state and fake terminal traffic; the workspace fixture also simulates agent
creation and machine-link responses. They refuse to run without `FLUTTER_TEST=1`, which disables
production-only pollers and persistence. The workspace fixture exercises the native macOS titlebar; its injected
Flutter keys do not establish physical AppKit keyboard/IME behavior. A fixture build replaces
`Harness.app`, so rebuild the normal review artifact afterward with
`bash scripts/build-macos-debug.sh --no-pub --target lib/main.dart`.

**Local macOS renderer:** Intel review builds need Skia, just like the Intel release.
Plain `flutter build macos --debug` leaves Impeller enabled and can produce invisible
bitmap artwork on Intel. The script above pins the built bundle's renderer and re-signs
it so Finder launches work too; Apple Silicon keeps Impeller. For `flutter run` and
native integration tests on Intel, add `--no-enable-impeller`. Check companion artwork
on the real renderer with `integration_test/companion_art_native_test.dart`; headless
image tests alone do not catch this failure.

Local stack / E2E scripts (the CLI comes from this repo's `../cli`; the backend from a sibling
`autonomous-code` checkout next to `autonomous-harness` — override with `AUTONOMOUS_CODE_ROOT` /
`HARNESS_REPO_ROOT`; see README):

```bash
make terminal-local-manual   # boots backend+CLI locally and runs lib/main_local_manual.dart
make terminal-local-e2e
make terminal-prod-e2e       # opt-in, refuses without PROD_TERMINAL_E2E=1 + release evidence vars
```

Release (`make release-desktop` from the repo root, which tags `vX.Y.Z_desktop` and lets CI build; `make upload-desktop VERSION=…` and
`make upload-desktop-linux` are the by-hand escape hatches, the latter on an Ubuntu host only) is
documented in RELEASE.md. **macOS ships TWO builds of one universal app**, differing only in

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [autonomous-ai/openharness](https://github.com/autonomous-ai/openharness) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
