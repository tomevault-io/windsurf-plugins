---
trigger: always_on
description: Copyright 2026 Google LLC
---

<!--
Copyright 2026 Google LLC

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    https://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
-->

# Agent Guide for Google Navigation for Flutter

This file provides instructions for AI coding agents working in this repository.
Read `README.md` and `CONTRIBUTING.md` before making substantial changes, and
follow existing code and test patterns when they are more specific than this
guide.

## Core rules

- Keep changes focused on the requested issue. Do not perform unrelated cleanup
  or broad refactoring without a clear need.
- Preserve existing public API behavior unless the task explicitly calls for a
  breaking change.
- Maintain Android and iOS parity for cross-platform features. Do not silently
  implement a public API on only one platform.
- Never hand-edit generated files. Change their source definitions and run the
  appropriate generator.
- Add or update tests for behavior changes and bug fixes.
- Update public documentation and the example app when user-facing behavior or
  API usage changes.
- Never commit API keys, credentials, signing material, or other secrets.
- Do not claim that checks passed unless they were actually run successfully.

## Repository layout

The repository contains one Flutter plugin and its example application:

- `lib/`: Public Dart API and plugin implementation.
  - `lib/google_navigation_flutter.dart`: Main public library export file.
  - `lib/src/`: Controllers, widgets, types, platform interfaces, method-channel
    code, and platform-specific Dart implementations.
- `pigeons/messages.dart`: Source of truth for the Pigeon platform-channel API.
- `android/`: Android plugin implementation and Kotlin unit tests.
- `ios/`: iOS plugin implementation and Swift package sources.
- `test/`: Dart unit tests, generated Pigeon tests, and generated mocks.
- `example/`: Example Flutter application, native iOS tests, and Patrol
  integration tests.
- `doc/`: Additional documentation and assets.
- `tool/`: Repository scripts, including the iOS native test runner.
- `.github/workflows/`: CI configuration and the current CI tool versions.

## Development setup

Use Flutter and Dart versions that satisfy `pubspec.yaml`. When reproducing a CI
failure, use the versions pinned in `.github/workflows/` rather than copying
version numbers into this file.

From the repository root, install the Melos version used by CI and bootstrap the
workspace. Replace `VERSION_FROM_CI` with the `melos-version` value in
`.github/workflows/test-and-build.yaml`:

```sh
dart pub global activate melos VERSION_FROM_CI
melos bootstrap
```

Additional tools depend on the files being changed:

- Android work requires the Java version configured by CI and a working Android
  SDK.
- iOS work requires macOS, the Xcode version used by CI, and `swift-format`.
- iOS uses Swift Package Manager; CocoaPods is not supported. Follow the SwiftPM
  setup in `README.md`.
- New source files may require the `addlicense` command described in
  `CONTRIBUTING.md`.
- Patrol is required only for integration tests. Use the Patrol CLI version
  configured by the integration-test workflow.

## Common commands

Run commands from the repository root unless noted otherwise.

```sh
# Static analysis. Warnings and infos are treated as failures.
melos run flutter-analyze

# Format Dart, Kotlin, and Swift sources.
melos run format

# Unit tests.
melos run test:dart
melos run test:android
melos run test:ios

# Release builds of the example app.
melos run flutter-build-android
melos run flutter-build-ios

# License checks.
melos run check-license-header
```

Before running native iOS tests on a fresh checkout, generate the Flutter iOS
configuration from `example/` (as CI does):

```sh
(cd example && flutter build ios --config-only)
```

The iOS test script accepts `TEST_DEVICE` and `TEST_OS` environment variables
when its defaults do not match the installed simulator runtime:

```sh
TEST_DEVICE='iPhone 17 Pro' TEST_OS='26.5' melos run test:ios
```

Treat the values above as examples. Use an installed simulator and the runtime
expected by the current CI workflow.

For a focused Dart test during development:

```sh
flutter test test/path/to/test_file.dart
```

CI's formatting validation script expects a clean checkout and fails when any
tracked file changes after formatting. In a normal working tree, run
`melos run format` and inspect `git diff` instead of treating that validation
script as a general-purpose local command.

## Generated code

### Pigeon messages

`pigeons/messages.dart` is the source of truth for platform-channel messages.
After changing it, run:

```sh
melos run generate:pigeon
```

This regenerates files including:

- `lib/src/method_channel/messages.g.dart`
- `android/src/main/kotlin/com/google/maps/flutter/navigation/messages.g.kt`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [googlemaps/flutter-navigation-sdk](https://github.com/googlemaps/flutter-navigation-sdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
