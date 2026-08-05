---
trigger: always_on
description: This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## Overview

Shark is a Swift command line tool with three subcommands:

- `generate` (default) — type-safe enums for iOS/macOS application assets (images, colors, fonts, localizations, storyboards, data assets). Parses the Xcode project to discover resources; compile-time safety for asset access.
- `lint` — localization completeness checks across all locales (missing-key, empty-source-value, placeholder-mismatch, orphaned-key); exit code 1 on findings, built as a CI gate.
- `translate` — fills localization gaps via local Claude Code by default, with Claude API and Codex CLI backends also supported; machine-validated format-specifier preservation and `needs_review` write-back.

## Positioning (keep docs/marketing consistent with this)

Shark's pitch against Xcode's generated symbols (which sherlocked images/colors in Xcode 15 and string accessors via String Catalogs later): Shark also covers fonts, storyboards, and data assets; speaks classic `.strings` *and* `.xcstrings`; namespaces by folder structure and key separators; handles multi-target/white-label setups (`--target`, `--exclude`, `--name`); and — the core argument — treats localization as a *workflow* (generate → lint → translate), not just codegen. Against SwiftGen/R.swift: project-driven instead of config-driven, zero runtime dependency, and they don't lint or translate. XcodeGen/Tuist are project generators — orthogonal, not competitors. The three subcommands deliberately share one project model and one `FormatSpecifierParser`, so claims like "lint checks exactly what generate generates" stay true — preserve that property when extending.

## Build and Development Commands

### Building the Project
```bash
# Build in debug mode
swift build

# Build for release
swift build -c release

# Install locally (after building release)
cp ./.build/release/Shark /usr/local/bin/shark
```

### Running Tests
The project uses Swift Package Manager's built-in testing. Look for test files in the standard `Tests/` directory structure.

```bash
swift test
```

### Running Smoke Tests
```bash
# Stable committed fixtures used by CI
Scripts/smoke-fixtures.sh

# Optional local pass over real-world projects
SHARK_REAL_WORLD_ROOT="$HOME/Documents/late" Scripts/smoke-real-world.sh
```

### Running the Tool
```bash
# Basic usage - generates Shark.swift in specified directory (generate is the default subcommand)
shark PROJECT_FILE_PATH OUTPUT_PATH

# Common usage with Xcode project
shark MyApp.xcodeproj ./Sources/MyApp/

# With options
shark MyApp.xcodeproj ./Sources/MyApp/ --target MyAppTarget --framework swiftui --name Assets

# Localization workflow
shark lint MyApp.xcodeproj --target MyAppTarget --format github
shark translate MyApp.xcodeproj --target MyAppTarget --to de,fr --dry-run
shark translate MyApp.xcodeproj --target MyAppTarget --to de,fr --backend claude-code --yes
shark translate MyApp.xcodeproj --target MyAppTarget --to de,fr --backend codex --yes
```

Field-tested on real multi-target projects; the prose-percent heuristics in the placeholder check (`"25% and"`, `"100%ig"`) and the `empty-source-value` rule came out of those runs — check `LocalizationLinterTests` before touching the normalization.

## Architecture

### Target Layout

The package is split into a library and a thin executable (see TRANSLATE_PLAN.md for the rationale — this boundary also serves the planned SPM build plugin, issue #46):

- **`SharkKit` (library)** — all logic: options, project parsing, codegen. Tests run against this target (`@testable import SharkKit`).
- **`Shark` (executable)** — ArgumentParser commands only. `Generate` is the `defaultSubcommand`, so the classic `shark PROJECT OUTPUT` invocation from Xcode build phases keeps working without naming a subcommand.

### Core Components

**CLI Layer (`Sources/Shark/`)**
- `Shark.swift`: Root command declaring the subcommands.
- `GenerateCommand.swift`: The codegen subcommand (default); orchestrates the generation process.
- `LintCommand.swift`: `shark lint` — missing-key / orphaned-key / placeholder-mismatch checks across all locales; exit code 1 on findings (CI gate).
- `TranslateCommand.swift`: `shark translate` — translates missing keys via local agent or API backend; backend selection (claude-code default, api, codex, auto), confirmation prompt with cost estimate.

**Localization Workflow (`Sources/SharkKit/Localization/`, `Lint/`, `Translate/`)**
- `LocalizationTable` + readers/writers: multi-locale model over `.strings` groups and `.xcstrings` catalogs. Writers are additive-only; the `.xcstrings` writer serializes in Xcode's JSON style (byte-identical round trip) to avoid whole-file diffs.
- `FormatSpecifierParser`: shared printf-specifier parsing used by codegen, lint, and translate validation.
- `LocalizationLinter` / `LintReportFormatter`: rules and text/json/github output.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kaandedeoglu/Shark](https://github.com/kaandedeoglu/Shark) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
