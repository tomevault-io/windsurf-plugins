---
trigger: always_on
description: This project maintains **separate version tracks** for macOS and iOS:
---

# Repository Guidelines

## Versioning & Release Process

This project maintains **separate version tracks** for macOS and iOS:

### Version Discovery

**CRITICAL: Always check the ACTUAL latest release before creating a new version.**

```bash
# Find the latest macOS release version:
gh release list --repo crmitchelmore/justspeaktoit | grep "mac-v" | head -1

# Or check GitHub releases page for mac-v* tags
# Example: if latest is mac-v0.9.1 → next should be mac-v0.9.2

# The VERSION file is NOT authoritative for releases - always check GitHub tags
```

### Tag Conventions

| Platform | Tag Format | Example | Workflow Triggered |
|----------|------------|---------|-------------------|
| macOS | `mac-v*` | `mac-v0.7.7` | `.github/workflows/release-mac.yml` |
| iOS | `ios-v*` | `ios-v0.9.1` | `.github/workflows/release-ios.yml` (manual) |
| Legacy | `v*` | `v0.7.5` | None (deprecated) |

### macOS Release Process

Releases are **fully automated** via conventional commits:

1. Push to `main` with a releasable commit type (`feat:`, `fix:`, `perf:`, or breaking change)
2. `auto-release.yml` determines the version bump and creates a `mac-v*` tag
3. `release-mac.yml` builds, notarises, publishes to GitHub Releases, updates appcast.xml, and updates Homebrew tap

Manual releases are still possible by creating and pushing a `mac-v*` tag directly.

### iOS Release Process

1. iOS uses **manual workflow dispatch** (not tag-triggered)
2. Go to Actions → "Release iOS (TestFlight)" → Run workflow
3. Enter version number (check App Store Connect for current version)

### VERSION File

The `VERSION` file is a **hint** used as fallback when no tag is present. It does NOT control the release version - the **tag determines the version**. Keep it updated but always verify against actual releases.

### Post-Release Support

When a user reports that a freshly shipped macOS release won't install or launch, perform the remediation directly when the environment permits filesystem/app actions; otherwise provide these exact steps for the user to run locally:
1. Download the latest `mac-v*` DMG.
2. Back up the existing app to a persistent location, e.g. `mv /Applications/JustSpeakToIt.app ~/Desktop/JustSpeakToIt.app.bak-$(date +%s)`.
3. Replace `/Applications/JustSpeakToIt.app` with the app from the DMG.
4. Verify launch locally (e.g. `open -n /Applications/JustSpeakToIt.app`) before asking the user to retry.
5. If local install verification is requested, confirm `/Applications/JustSpeakToIt.app` exists, read `CFBundleShortVersionString`, and verify the process is running.

## Project Structure & Module Organization

This project uses **Swift Package Manager** for modularization with cross-platform support:

```
Package.swift           # Defines all targets and dependencies
Sources/
├── SpeakCore/          # Shared cross-platform library (types, protocols, keychain)
├── SpeakApp/           # macOS application (executable)
└── SpeakiOS/           # iOS library (views, services, with #if os(iOS) guards)
SpeakiOSApp/            # iOS app entry point (@main)
Project.swift           # Tuist manifest (Xcode project generation)
Workspace.swift         # Tuist workspace manifest
Just Speak to It.xcodeproj/ # Generated Xcode project
Tests/                  # XCTest suite
```

### Swift Package Targets

| Target | Type | Platform | Description |
|--------|------|----------|-------------|
| `SpeakCore` | Library | macOS + iOS | Cross-platform types, protocols, secure storage |
| `SpeakApp` | Executable | macOS | macOS SwiftUI application |
| `SpeakiOSLib` | Library | iOS | iOS views and services (exported with `public` APIs) |

### Modularization Patterns

1. **Shared code in SpeakCore**: Types, protocols, and utilities that work on both platforms
2. **Platform guards**: Use `#if os(iOS)` / `#if os(macOS)` for platform-specific code
3. **Public APIs for libraries**: Types in `SpeakiOSLib` must be `public` for Xcode project to access
4. **Tuist links packages**: `Project.swift` references the local Swift package for Xcode generation

### iOS App Structure

The iOS app is built via Xcode but sources come from Swift packages:
- `SpeakiOSApp/SpeakiOSApp.swift` - Entry point with `@main`
- Links `SpeakCore` and `SpeakiOSLib` as package dependencies
- Run `tuist generate` and open `"Just Speak to It.xcworkspace"` in Xcode to build/run on device

## Build, Test, and Development Commands

### macOS (SwiftPM)
- `make` or `make run` – Build and launch the macOS app
- `make build` – Debug compilation only
- `make rebuild` – Clean and rebuild from scratch
- `make test` – Execute XCTest suite
- `swift build --target SpeakiOSLib` – Verify iOS library compiles

### iOS (Xcode)
- `tuist generate` – Generate the Xcode workspace
- `open "Just Speak to It.xcworkspace"` – Open in Xcode
- Select device/simulator and build (Cmd+B)
- Or use xcodebuild MCP for automation (see below)

### Linting
```bash
swift package plugin --allow-writing-to-package-directory swiftlint --strict --target SpeakApp
swift package plugin --allow-writing-to-package-directory swiftformat --target SpeakApp
```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [crmitchelmore/justspeaktoit](https://github.com/crmitchelmore/justspeaktoit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-08 -->
