---
trigger: always_on
description: - This repository is a Flutter app named `bilimusic`.
---

# AGENTS.md

## Purpose
- This repository is a Flutter app named `bilimusic`.
- Stack: Flutter, Riverpod code generation, Dio, Hive CE, Freezed, GoRouter.
- Current app focus: Bilibili QR-code authentication and local session persistence.
- Use this file as the default operating guide for coding agents working in this repo.

## Project Layout
- `lib/main.dart`: app bootstrap and global initialization.
- `lib/myApp.dart`: root widget and app-level theme/router wiring.
- `lib/router/`: navigation setup via GoRouter.
- `lib/core/`: shared infrastructure such as networking, theme, and Hive setup.
- `lib/feature/auth/`: feature-layered auth code split into `data`, `domain`, `logic`, and `ui`.
- `test/`: Flutter tests. At the moment there is only `test/widget_test.dart`.
- `docs/`: reference material, including Bilibili API documentation snapshots.

## Commands

Before running any `flutter` or `dart` command, first ensure the command is executed from the project root directory for this repository.

### Setup
- Install dependencies: `flutter pub get`
- Check Flutter environment: `flutter doctor`

### Run
- Run on default device: `flutter run`
- Run on a specific device: `flutter run -d <device-id>`
- Run web target: `flutter run -d chrome`

### Build
- Android APK: `flutter build apk`
- Android app bundle: `flutter build appbundle`
- iOS: `flutter build ios`
- Web: `flutter build web`
- Linux/macOS/Windows: use the corresponding `flutter build <platform>` command if that platform is enabled.

### Lint and Format
- Static analysis: `flutter analyze`
- Format only Dart files changed for the current task, for example `dart format lib/feature/auth/ui/login_page.dart test/widget_test.dart`.
- Do not format the entire repository unless the user explicitly asks for it.

### Tests
- Run all tests: `flutter test`
- Run one test file: `flutter test test/widget_test.dart`
- Run a single named test: `flutter test --plain-name "Counter increments smoke test" test/widget_test.dart`
- Run tests with machine output: `flutter test --machine`

### Code Generation
- One-off codegen: `dart run build_runner build --delete-conflicting-outputs`
- Watch mode: `dart run build_runner watch --delete-conflicting-outputs`

### Recommended Validation Flow After Changes
- `flutter pub get` if dependencies changed.
- `dart format <changed Dart files only>`.
- `flutter analyze`
- `flutter test`
- `dart run build_runner build --delete-conflicting-outputs` if any `@riverpod`, `@freezed`, Hive adapters, or `part` files changed.

## Single-Test Guidance
- Prefer `flutter test <path>` when validating one test file.
- Prefer `--plain-name` when validating one case inside a file.
- Example for a future auth test: `flutter test --plain-name "starts QR login" test/feature/auth/logic/bili_auth_controller_test.dart`
- There are no `integration_test/` files in the repo right now.

## Source Control Expectations
- Do not edit generated files unless the task explicitly requires it.
- Generated files in this repo include `*.g.dart` and `*.freezed.dart`.
- When source changes affect generated output, update the source file first, then rerun build_runner.
- Keep unrelated workspace changes intact; this repo may already be dirty.

## Existing Agent Rules Check
- No `.cursor/rules/` directory was found.
- No `.cursorrules` file was found.
- No `.github/copilot-instructions.md` file was found.
- As a result, this file is the primary agent instruction source inside the repository.

## Architecture Conventions
- Follow the existing feature-first structure under `lib/feature/<feature-name>/`.
- Within a feature, keep responsibilities split by layer:
- `domain`: enums, immutable models, simple state objects, value types.
- `data`: repository and persistence/network translation logic.
- `logic`: Riverpod providers, notifiers, orchestration, polling, state transitions.
- `ui`: widgets, view composition, widget-only presentation logic.
- Put reusable infrastructure in `lib/core/` rather than inside one feature.
- Keep routing centralized in `lib/router/`.

## Imports
- Use package imports for app code, for example `package:bilimusic/...`.
- Keep Dart SDK imports first, then package imports, then relative imports if absolutely needed.
- Prefer package imports over long relative traversals.
- Keep `part` directives after imports and before declarations.
- Let `dart format` manage wrapping; do not hand-align imports.

## Formatting
- Use `dart format`; do not preserve custom spacing that conflicts with formatter output.
- Prefer trailing commas in multi-line widget trees, argument lists, and constructors.
- Keep lines formatter-friendly instead of fighting wrapping.
- Use blank lines to separate logical sections, not every small statement.
- Avoid decorative comments and commented-out code.

## Types and Null Safety
- The codebase uses sound null safety; preserve it.
- Prefer explicit types when they improve readability or match existing local style.
- Existing code often annotates locals and fields explicitly; that is a safe default.
- Use precise generic types such as `Map<String, dynamic>` and `Box<String>`.
- Avoid `dynamic` unless the external API truly requires it.
- Convert loosely typed API responses at the boundary, then work with typed values.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AprDeci/bili-music](https://github.com/AprDeci/bili-music) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
