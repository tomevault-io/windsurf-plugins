---
trigger: always_on
description: Guidance for any AI coding agent working in this repository. Humans should read [`CONTRIBUTING.md`](CONTRIBUTING.md) and [`docs/README.md`](docs/README.md) instead — those are the sources of truth for contributor workflow and where docs live.
---

# AGENTS.md

Guidance for any AI coding agent working in this repository. Humans should read [`CONTRIBUTING.md`](CONTRIBUTING.md) and [`docs/README.md`](docs/README.md) instead — those are the sources of truth for contributor workflow and where docs live.

If your harness loads a tool-specific file (such as `CLAUDE.md`), that file is a stub that points back here.

## Project Overview

macOS SwiftUI app that exports the Apple Photos library to local/external storage in an organized folder hierarchy. Uses system frameworks only (no CocoaPods/SwiftPM dependencies). Targets macOS 15.x with Xcode 16.x.

## Build & Test Commands

```bash
# Build (Debug)
xcodebuild -project photo-export.xcodeproj -scheme "photo-export" -configuration Debug -destination 'platform=macOS' CODE_SIGNING_ALLOWED=NO build

# Run all unit tests
xcodebuild -project photo-export.xcodeproj -scheme "photo-export" -destination 'platform=macOS' CODE_SIGNING_ALLOWED=NO test

# Run a single test class
xcodebuild -project photo-export.xcodeproj -scheme "photo-export" -destination 'platform=macOS' -only-testing:photo-exportTests/ExportRecordStoreTests CODE_SIGNING_ALLOWED=NO test

# Lint (use a workspace-local cache if your sandbox blocks ~/Library)
swiftlint --strict
swiftlint --strict --cache-path build/swiftlint-cache  # sandboxed alternative

# Format check / auto-fix
swift-format lint --recursive photo-export
swift-format format --recursive --in-place photo-export
```

UI tests exist in `photo-exportUITests/` but are skipped by default in the shared scheme.

## Architecture

**Pattern:** SwiftUI + Managers. Views are thin; logic lives in Managers and ViewModels.

Source code under `photo-export/` is organized as follows:

- `Managers/` — long-lived stateful services and pure helpers (see breakdown below)
- `Protocols/` — test seams: `PhotoLibraryService`, `AssetResourceWriter`, `FileSystemService`, `ExportDestination`. Add a new protocol here when you need to inject a fake.
- `Models/` — value types: `AssetDescriptor`, `AssetDetails`, `ExportRecord`, `ExportVariant`, `ExportPlacement`, `LibrarySelection`, `PhotoCollectionDescriptor`
- `Views/` — SwiftUI views (see list below)
- `ViewModels/` — `MonthViewModel`
- `Helpers/` — small pure utilities (`MonthFormatting`)
- `Resources/`, `SupportingFiles/`, `Assets.xcassets` — bundle resources, Info.plist, asset catalog

**App entry point** (`photo_exportApp.swift`): creates five `@StateObject` dependencies and injects them as `@EnvironmentObject` into the view hierarchy:

- **PhotoLibraryManager** — Photos framework authorization and asset fetching (thumbnails, full-size images). Uses `PHCachingImageManager`.
- **ExportDestinationManager** — manages the chosen export destination folder (security-scoped bookmarks).
- **ExportRecordStore** — tracks timeline (year/month) exports per-destination. Reconfigures when destination changes.
- **CollectionExportRecordStore** — sibling store for Favorites + user-album exports per-destination. Disjoint key space from the timeline store; the two stores cannot corrupt each other. Routed to by `ExportManager` via `placement.kind`.
- **ExportManager** — orchestrates the export queue (enqueue/pause/cancel/resume). Depends on the other four managers; routes record mutations to the correct store via `ExportPlacement.kind`.

**Other code under `Managers/`:**

- `BackupScanner` — scans an existing backup folder and matches files to Photos assets (used by Import Existing Backup)
- `ExportFilenamePolicy` — pure rules for `_orig` companion filenames
- `ExportPathPolicy` — pure path-component sanitization for collection folder names
- `ExportPlacementResolver` — maps a `LibrarySelection` to an `ExportPlacement`, including sibling-collision suffixing for albums
- `CollectionCountCache` — actor that dedups concurrent count fetches for the Collections sidebar; invalidated on `PHPhotoLibraryChangeObserver` callbacks
- `JSONLRecordFile` — shared JSONL+snapshot persistence used by both record stores
- `ExportRecordsDirectoryCoordinator` — runs the legacy `<oldId>` → `<newId>` directory migration once before either store configures
- `ResourceSelection` — picks the byte source for an edited variant via the `EditedProducer` enum
- `ProductionAssetResourceWriter` — production implementation behind the `AssetResourceWriter` seam
- `ProductionMediaRenderer` — production implementation behind the `MediaRenderer` seam (edited videos)
- `FileIOService` — atomic file moves and timestamp handling (conforms to `FileSystemService`)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [valtteriluomapareto/photo-export](https://github.com/valtteriluomapareto/photo-export) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
