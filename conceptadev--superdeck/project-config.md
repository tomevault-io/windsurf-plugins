---
trigger: always_on
description: This file provides guidance to Claude Code and other AI assistants working on the SuperDeck codebase.
---

# AGENTS.md

This file provides guidance to Claude Code and other AI assistants working on the SuperDeck codebase.

> **Note**: `CLAUDE.md` is a symlink to this file.

## Project Overview

SuperDeck is a Flutter presentation framework that renders slides written in Markdown. Users write slides in a `slides.md` file using Markdown syntax with custom block annotations, and SuperDeck renders them as a Flutter application.

- **Live demo**: https://superdeck-dev.web.app
- **Repository**: https://github.com/conceptadev/superdeck

## Project Structure

This is a Melos monorepo with the following packages:

```
packages/
  core/       # Shared deck models, schemas, Markdown utilities, storage contracts (Dart-only)
  superdeck/  # Flutter widgets and presentation components
  cli/        # superdeck CLI tool (setup, build/watch, version)
  builder/    # Deck parsing, serialization, and build/watch pipeline (Dart-only)
  playground/ # SuperDeck Wizard: AI deck generation, presenting, exporting
  plugins/pdf/ # PDF export plugin
demo/         # Sample presentation app
docs/         # User-facing documentation (MDX format)
```

### Key Package Responsibilities

- **core**: Shared Markdown utilities, slide/block configuration, model/schema validation, storage contracts, and YAML utilities (no Flutter dependency)
- **superdeck**: Flutter widgets, DeckController, navigation, thumbnail/capture services, theme system
- **cli**: CLI commands for project setup and building slides
- **builder**: Markdown deck parsing, serialization, build/watch orchestration, and build plugins
- **playground**: AI deck generation (the Wizard), present mode, and deck export
- **plugins/pdf**: PDF capture and export UI

## Environment Setup

This project uses FVM (Flutter Version Management) configured via `.fvmrc` (pinned to Flutter `3.47.5`):

```bash
fvm use --force
fvm dart run melos bootstrap
```

Always work inside the FVM-provided SDK (`.fvm/flutter_sdk`) to avoid toolchain drift.

**Required SDK versions**: Dart >=3.12.0, Flutter >=3.44.6

## Common Commands

### Analysis & Linting
```bash
fvm dart run melos run analyze          # Run dart analyze + DCM analysis
fvm dart run melos run analyze:all      # Full analysis including unused code/files
fvm dart run melos run fix              # Apply dart fix + DCM autofixes
```

### Code Generation
```bash
fvm dart run melos run build_runner:build   # Generate code (run before tests)
fvm dart run melos run build_runner:watch   # Watch mode for development
fvm dart run melos run build_runner:clean   # Clean generated files
fvm dart run melos run brb                  # Alias for build_runner:build
fvm dart run melos run brbc                 # Alias for build_runner:clean
```

### Testing

Each command covers one layer. No single command covers every layer.

```bash
fvm dart run melos run test               # Package unit and widget tests
fvm dart run melos run test:integration   # Desktop integration tests (Linux)
fvm dart run melos run test:integration:macos  # Desktop integration tests (macOS)
fvm dart run melos run test:e2e:web       # Browser smoke tests (Chromium, WebKit)
fvm dart run melos run test:e2e           # Linux integration + browser smoke
fvm dart run melos run test:all           # Package tests + Linux integration tests
fvm dart run melos run test:coverage      # Package tests with coverage
fvm flutter test <path>                   # One test file
```

`melos run test` excludes `ci-excluded` suites; `test:coverage` includes them.
No melos command runs the live generation tests. Run those from
`packages/playground`:

```bash
# Deterministic checkpoint, no provider call.
fvm flutter test test_live/ai_generation/ai_generation_smoke_test.dart \
  --dart-define=LIVE_FAKE_CHECKPOINT=true --reporter expanded

# Live 10-slide smoke test; requires the repository-root .env file.
fvm flutter test test_live/ai_generation/ai_generation_smoke_test.dart \
  --dart-define=LIVE_FIXTURE=superdeck_demo_10 \
  --dart-define-from-file=../../.env --reporter expanded
```

The live cases skip when the supplied defines contain no `GOOGLE_AI_API_KEY`.
A missing define file fails before the tests start. Omitting `LIVE_FIXTURE`
selects the default small-fixture suite rather than the 10-slide checkpoint.

### Running Apps & Live Debugging
```bash
cd packages/playground
fvm flutter run -d macos -t lib/main.dart --dart-define-from-file=../../.env
```

The playground reads `GOOGLE_AI_API_KEY` from the ignored repository-root
`.env` file through Flutter's compile-time define-file option. Without that
flag, the Wizard intentionally shows a configuration error before accepting
input.

Wizard image generation is enabled by default for local/debug demos and is
opt-in for release builds. Validate the image-enabled smoke flow before adding
`--dart-define=SUPERDECK_WIZARD_IMAGE_GENERATION=true` to a release launch.

When running an app to reproduce or diagnose a UI/runtime issue, launch it with
`fvm flutter run` and keep that process attached for the entire reproduction.
Continue reading its output after each UI interaction so Dart exceptions,
Flutter framework errors, plugin failures, and native macOS logs are captured as

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [conceptadev/superdeck](https://github.com/conceptadev/superdeck) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
