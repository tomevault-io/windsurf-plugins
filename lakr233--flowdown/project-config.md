---
trigger: always_on
description: This file provides guidance to AI coding agents working inside this repository.
---

# FlowDown Agent Guide

This file provides guidance to AI coding agents working inside this repository.

## Overview

FlowDown is a Swift-based AI/LLM client for iOS and macOS (Catalyst) with a privacy-first mindset. The workspace hosts the main app plus several Swift Package Manager frameworks (e.g. `ChatClientKit`, `Storage`, `Logger`) that power storage, editing, model integrations, and on-device MLX inference. Persistent configuration lives in the `ConfigurableKit` package.

- All code text (UI strings, comments, logs) must remain in English.

## Environment & Tooling

- Prefer opening `FlowDown.xcworkspace` so the app and frameworks resolve together under shared schemes.
- `ChatClientKit` intentionally relies on the `FlowDown.xcworkspace` package override for `mlx-swift-lm`; keep `Frameworks/ChatClientKit/Package.swift` on `branch: "main"` for that dependency and validate integration changes through workspace builds driven by the top-level `Makefile`.
- Use Xcode 26.x (Swift 6.0 toolchain) or newer to satisfy package manifests and the Swift `Testing` library.
- Build on macOS 26 or later to ensure compatibility with the required toolchain.
- Install `xcbeautify` (`brew install xcbeautify`) so the shared `make` workflows can produce readable logs.
- Lean on automation in `Resources/DevKit/scripts/` (localization, archiving, licensing) instead of ad-hoc scripts.
- Always use the top-level `Makefile` for build, test, package resolution, archive, and verification flows.
- Do not run `xcodebuild` directly in the shell. If a workflow is missing, add a `Makefile` target first.

## Platform Requirements & Dependencies

- Target platforms reflect framework minimums: iOS 17.0+, macCatalyst 17.0+ (macOS 14+ for Catalyst helpers).
- Toolchain: Swift 6.0 (`swift-tools-version: 6.0`) and the Xcode 26 SDK line are required. MLX currently resolves to `mlx-swift` 0.21.x and `mlx-swift-examples` on `main`.
- Core SwiftPM dependencies include MLX/MLX examples, ConfigurableKit, SnapKit, SwifterSwift, MarkdownView, WCDB prebuilt binaries, ZIPFoundation, ScrubberKit, AlertController, GlyphixTextFx, ColorfulX, UIEffectKit, DpkgVersion, swift-transformers, and additional UI/tooling libraries listed in `FlowDown.xcodeproj`.
- `Storage` wraps WCDB with Markdown parsing and ZIP export; `ChatClientKit` layers MLX, EventSource, and Logger to deliver on-device and streaming chat.
- MLX GPU support is automatically detected and disabled in simulator/x86_64 builds (see `FlowDown/main.swift`).

## Project Structure

- `FlowDown.xcworkspace`: Entry point with app and frameworks.
- `FlowDown/`: Application sources divided into `Application/` (entry surfaces), `Backend/` (conversations, models, storage, security), `Interface/` (UIKit), `PlatformSupport/` (macOS/Catalyst glue), and `BundledResources/` (curated assets shipped with the app).
- `FlowDown/DerivedSources/`: Generated during builds (`BuildInfo.swift`, `CloudKitConfig.swift`). Treat as generated—schemes will overwrite changes.
- `Frameworks/`: Shared Swift packages (`ChatClientKit`, `Storage`, `RichEditor`, `RunestoneEditor`, `Logger`). Each package owns its manifest and dependency graph.
- `FlowDownUnitTests/`: App-level tests using Swift's `Testing` package (`@Test` entry points).
- `Resources/`: Shared assets, localization collateral, privacy documents, and DevKit utilities.
- `Resources/DevKit/scripts/`: Automation helpers (archiving, translation, licence scanning). Prefer extending these over new stand-alone scripts.
- `Playgrounds/`: Exploratory prototypes; do not assume production readiness.

## Build & Run Commands

- Open the workspace: `open FlowDown.xcworkspace`.
- Build commands:
  - `make build` for the iOS and Mac Catalyst app
  - `make build-ios` for the iOS app
  - `make build-catalyst` for the Mac Catalyst app
  - `make build-extension` for the translation provider extension
- Test commands:
  - `make test` for the default test flow
  - `make test-unit` for app tests on the first available iOS simulator
  - `make test-online-e2e` for the online E2E suite
- Package and license commands:
  - `make package-resolve` to resolve SwiftPM packages
  - `make scan-license` to refresh `OpenSourceLicenses.md`
- Localization commands:
  - `make localization-check` to check for missing translations
  - `make localization-stale-check` to prune stale keys and verify completeness
- Archive commands:
  - `make archive` for both platforms
  - `make archive-ios` for the iOS archive
  - `make archive-macos` for the macOS archive
- Cleanup commands:
  - `make clean-build` to remove repo-local build artifacts
  - `make clean` to remove repo-local build artifacts and derived data
- Archive script automatically commits changes and bumps version before building; ensure the working tree is clean beforehand.
- Use `make help` to discover the current command surface.
- Localization validation helpers:
  - `make localization-stale-check`
  - `make localization-check`
  - `python3 Resources/DevKit/scripts/update_missing_i18n.py FlowDown/Resources/Localizable.xcstrings` to scaffold missing locales; extend `NEW_STRINGS` in that script when adding new keys.

## Shell Script Style

### Core Principles

- **Simplicity**: Keep scripts minimal and focused

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Lakr233/FlowDown](https://github.com/Lakr233/FlowDown) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
