---
trigger: always_on
description: This file provides guidance to AI coding assistants when working with code in this repository.
---

# AI Assistant Guide

This file provides guidance to AI coding assistants when working with code in this repository.

## Guiding Principles (MUST FOLLOW)

- **Keep it clear**: Write Swift code that is easy to read, maintain, and explain. Prefer clarity over cleverness.
- **Match the house style**: Reuse existing patterns, naming, and conventions found in the codebase.
- **Search smart**: Use glob/grep for codebase exploration before making assumptions about structure or patterns.
- **Log centrally**: Route all logging through `AppLogger.shared` with the right context—never use `print` in production code.
- **Always propose before executing**: Before making any changes, clearly explain your planned approach and wait for explicit user approval.
- **Build and test before completion**: Coding tasks are only complete after a successful `xcodebuild clean build` (Release). If the change touches a tested module, run the corresponding test suite.
- **Write conventional commits**: Commit small, focused changes using Conventional Commit messages (`feat:`, `fix:`, `docs:`, `refactor:`, `test:`).
- **Build after every code change**: Verify compilation succeeds before delivering any code change.

## Project Overview

AppPorts is a native macOS desktop app (Swift/SwiftUI) that migrates applications from `/Applications` to external storage while keeping a functional local "portal" via Stub Portal (launcher script). It also supports migrating data directories (`~/Library/` subfolders and dot-folders like `~/.npm`) and user-selected custom folders. Minimum deployment target: macOS 12.0 (Monterey).

## Build & Test Commands

If Xcode is installed outside `/Applications`, export `DEVELOPER_DIR` before running `xcodebuild`:
```bash
export DEVELOPER_DIR=/Volumes/hano/Applications/Xcode.app/Contents/Developer
```

**Build** (Xcode project, no SPM):
```bash
xcodebuild clean build -scheme "AppPorts" -configuration Release -destination 'platform=macOS' \
  CODE_SIGN_IDENTITY="" CODE_SIGNING_REQUIRED=NO CODE_SIGN_ENTITLEMENTS="" CODE_SIGNING_ALLOWED=NO
```

**Run all tests:**
```bash
xcodebuild test -scheme "AppPorts" -configuration Debug -destination 'platform=macOS' \
  CODE_SIGN_IDENTITY="" CODE_SIGNING_REQUIRED=NO CODE_SIGN_ENTITLEMENTS="" CODE_SIGNING_ALLOWED=NO
```

**Run a single test class** (e.g. `DataDirScannerTests`):
```bash
xcodebuild test -scheme "AppPorts" -destination 'platform=macOS' \
  -only-testing:"AppPortsTests/DataDirScannerTests" \
  CODE_SIGN_IDENTITY="" CODE_SIGNING_REQUIRED=NO CODE_SIGNING_ALLOWED=NO
```

**Run localization audit:**
```bash
xcodebuild test -scheme "AppPorts" -destination 'platform=macOS,arch=arm64' \
  -only-testing:"AppPortsTests/LocalizationAuditTests" \
  CODE_SIGNING_ALLOWED=NO -derivedDataPath /tmp/AppPortsDerived
```

### Test Suites Overview

| Test Suite | Module | When to Run |
|------------|--------|-------------|
| `DataDirMoverTests` | Data directory migration | When touching `DataDirMover` |
| `DataDirScannerTests` | Data directory scanning | When touching `DataDirScanner` |
| `CustomDirScannerTests` | Custom directory configs/scanning/validation | When touching `CustomDirModels`, `CustomDirScanner`, or custom directory UI |
| `AppMigrationServiceTests` | App migration | When touching `AppMigrationService` |
| `AppScannerTests` | App scanning | When touching `AppScanner` |
| `AppLoggerTests` | Logging & diagnostics | When touching `AppLogger` |
| `LocalizationAuditTests` | Localization | When touching user-facing copy |
| `UpdateCheckerTests` | Release/update lookup | When touching `UpdateChecker` |

## Architecture

### Directory Structure

```
AppPorts/
├── AppPorts.xcodeproj/             # Xcode project (no SPM, no external deps)
├── AppPorts/                       # Main source
│   ├── Appports.swift              # @main entry point + AppDelegate
│   ├── ContentView.swift           # Main window and top-level tab routing
│   ├── WelcomeView.swift           # First-launch welcome screen
│   ├── AboutView.swift             # About dialog with GitHub contributors
│   ├── Localizable.xcstrings       # String catalog (20+ languages)
│   ├── Models/
│   │   ├── AppModels.swift         # AppItem, AppMoverError, AppContainerKind
│   │   ├── DataDirItem.swift       # DataDirItem, DataDirType, DataDirPriority
│   │   ├── CustomDirModels.swift   # CustomDirConfig, validation, display entries
│   │   └── AppLanguageOption.swift # Language catalog (AppLanguageCatalog)
│   ├── Views/
│   │   ├── DataDirsView.swift      # Tool dirs + app data management
│   │   ├── CustomDirsView.swift    # User-selected folder migration tab
│   │   ├── AppStoreSettingsView.swift # Settings sheet
│   │   └── Components/
│   │       ├── AppIconView.swift   # Async app icon loader
│   │       ├── AppRowView.swift    # App list row + context menu
│   │       ├── DataDirRowView.swift # Data dir row + tree indentation
│   │       ├── CustomDirRowView.swift # Custom folder row
│   │       ├── StatusBadge.swift   # Status pill badges (link/framework/type)
│   │       ├── ProgressOverlay.swift # Migration progress overlay
│   │       └── HelpButton.swift    # Popover help button
│   ├── Services/
│   │   ├── AppMigrationService.swift # Core migration engine (~1500 lines)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wzh4869/AppPorts](https://github.com/wzh4869/AppPorts) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
