---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Arcmark is a macOS bookmark management application built with Swift and AppKit. It provides a workspace-based organization system for links and folders with features like drag-and-drop, inline editing, and automatic favicon/title fetching.

## Development Commands

**IMPORTANT**: Never run `./scripts/run.sh` or any build/run scripts automatically. The user will run these scripts in the background when needed.

### Building and Testing
```bash
# Development builds (ad-hoc signing)
./scripts/build.sh                  # Build app only
./scripts/build.sh --dmg            # Build app and create DMG

# Production builds (Developer ID + notarization)
./scripts/build.sh --production     # Build with Developer ID signing
./scripts/build.sh --production --dmg  # Build and create notarized DMG

# Release (build + sign + tag + push + GitHub release)
./scripts/release.sh 0.2.0         # Full release
./scripts/release.sh 0.2.0 --dry-run  # Build only, skip git/GitHub

# Other commands
./scripts/create-dmg.sh             # Create DMG from existing build
./scripts/run.sh                    # Build and run the app

# Testing
swift test                          # Run all tests
swift test --filter ModelTests.testJSONRoundTrip  # Run single test

# Library build (Swift PM only)
swift build -c release
```

**Build System:** The app uses Swift Bundler to create macOS app bundles. The build script automatically:
- Reads version from `VERSION` file and syncs to Bundler.toml
- Builds the app with Swift Bundler
- Patches Info.plist to ensure CFBundleIdentifier is present
- Code signs the app (ad-hoc for development, Developer ID for production)
- Verifies the build
- Optionally creates a DMG installer with `--dmg` flag
- Optionally notarizes the DMG with `--production --dmg`

**Production Signing**: For distribution outside the Mac App Store, use `--production` flag with proper code signing credentials configured in `.notarization-config`. See [docs/PRODUCTION_SIGNING.md](docs/PRODUCTION_SIGNING.md) for setup.

See [docs/BUILD_AND_CODESIGN.md](docs/BUILD_AND_CODESIGN.md) for detailed information about the build process, code signing, and verification.

### Version Management

The app version is managed through a centralized `VERSION` file in the project root. To update the version:

```bash
echo "0.2.0" > VERSION
```

The build script automatically reads this file and updates `Bundler.toml` and Info.plist accordingly. The version follows [Semantic Versioning](https://semver.org/): `MAJOR.MINOR.PATCH`.

For complete distribution workflow including DMG creation and beta testing, see [docs/DISTRIBUTION.md](docs/DISTRIBUTION.md).

## Architecture

### Data Flow Architecture

The application follows a unidirectional data flow pattern:

1. **AppModel** - Central state manager that owns `AppState` and coordinates all mutations
   - Single source of truth for application state
   - Exposes an `onChange` callback for UI updates
   - All state mutations go through AppModel methods (no direct state manipulation)
   - Automatically persists to disk via `DataStore` after every mutation

2. **DataStore** - Handles persistence layer
   - Saves/loads JSON to `~/Library/Application Support/Arcmark/data.json`
   - Manages favicon storage in `Icons/` subdirectory
   - Provides default state initialization

3. **MainViewController** - Primary UI controller
   - Observes AppModel via `onChange` callback
   - Manages NSCollectionView for hierarchical node display
   - Handles drag-and-drop, inline editing, and context menus
   - Never directly mutates state - always calls AppModel methods

### Core Data Model

The data model is defined in `Models.swift`:

- **AppState** - Root container holding workspaces and selected workspace ID
- **Workspace** - Named container with emoji, color, and hierarchical items
- **Node** - Enum representing either a `Folder` or `Link`
  - `Folder` - Contains nested children and isExpanded state
  - `Link` - URL, title, and optional favicon path

All models are Codable and use UUID-based identification. The Node enum uses custom encoding to serialize the tagged union structure.

### Key Components

**Services** (all MainActor singletons):
- **FaviconService** - Async favicon fetching with disk caching and failure cooldown
- **LinkTitleService** - HTML title extraction from URLs
- **BrowserManager** - Manages browser selection and URL opening

**UI Components**:
- **MainViewController** - Collection view-based hierarchical list with animations (reduced from 1352 to 596 lines through Phase 3 refactoring)
- **NodeListViewController** - Manages collection view, drag-drop, context menus (extracted from MainViewController)
- **SearchCoordinator** - Handles search/filtering logic (extracted from MainViewController)
- **WorkspaceManagementView** - Manages workspace list in settings
- **NodeCollectionViewItem** - Reusable cell with icon, title, hover states, delete button
- **SearchBarView** - Search field that filters nodes
- **IconTitleButton** - Custom button for paste action
- **ListFlowLayout** - Custom NSCollectionViewLayout for vertical list


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Geek-1001/arcmark](https://github.com/Geek-1001/arcmark) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
