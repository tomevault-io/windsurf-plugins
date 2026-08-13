---
trigger: always_on
description: macOS SwiftUI app for automated camera card backup. Single copy engine: the **FilmCan Engine** (`CustomCopierService`, the fan-out copier) — Swift-only. The rsync engine (code + UI) was removed in 1.2.x; `RsyncOptions`/`CopyEngine` models replaced by `EngineOptions`/`DefaultExcludes` in 1.3.x. Note: the build still bundles the rsync binary **only** because its libs ship `libxxhash.0.dylib`, which `XXHash.swift` dlopen's for xxh128 verification (`Resources/rsync/lib/<arch>/`). Dropping the rsync 
---

# FilmCan — Agent Instructions

## What This Is

macOS SwiftUI app for automated camera card backup. Single copy engine: the **FilmCan Engine** (`CustomCopierService`, the fan-out copier) — Swift-only. The rsync engine (code + UI) was removed in 1.2.x; `RsyncOptions`/`CopyEngine` models replaced by `EngineOptions`/`DefaultExcludes` in 1.3.x. Note: the build still bundles the rsync binary **only** because its libs ship `libxxhash.0.dylib`, which `XXHash.swift` dlopen's for xxh128 verification (`Resources/rsync/lib/<arch>/`). Dropping the rsync binary requires vendoring just libxxhash first (tracked in technical-debt). macOS 13+, Swift 5.9, Xcode 15+.

## Quick Start

```
open FilmCan/FilmCan.xcodeproj   # XcodeGen-generated — do not edit the .xcodeproj directly
```

After editing `project.yml`: `xcodegen generate` then reopen.

## Project Structure

```
FilmCan/Sources/
├── App/             # FilmCanApp.swift (@main entry), MainView, SettingsView
├── Views/           # All SwiftUI views (tabs: Backup, History, Settings, About)
├── ViewModels/      # ObservableObjects, especially TaskState
├── Services/        # CustomCopierService, NotificationService, WebhookService, etc.
├── Models/          # DriveInfo, TaskResult, Preset, HistoryEntry, etc.
├── Utilities/       # Logger, HashListGenerator, DiskArbitration helpers
└── Resources/       # Assets, entitlements
```

Local data lives in `~/Application Support`.

`FilmCan/build/` (DerivedData) and `FilmCan/dist/` (staged .app, DMG) are local build artifacts — exclude them from searches.

## Key Commands

| Action | Command |
|--------|---------|
| Build/Run | Open `.xcodeproj` → Cmd+R |
| Regenerate project | `xcodegen generate` (run in `FilmCan/`) |
| Release build | `FilmCan/scripts/package_release.sh` (universal binary + customized DMG) |
| Run tests | Xcode Cmd+U (`FilmCanTests` target). No shared scheme is committed — `xcodebuild test` only works after Xcode auto-generates one |
| QA checklist | `docs/qa.md` + `docs/smoke-qa-checklist.md` (manual passes, complement the automated suite) |

## Architecture Notes

- **Single copy engine**: `CustomCopierService` (the FilmCan Engine — Swift-only, no external deps), driving the `FanOutCopier` actor. Supports pause/resume and duplicate detection. The rsync engine (`RsyncService`) was fully removed in 1.2.x — nothing shells out to rsync at runtime. **Build-time**, Homebrew rsync ≥ 3.4.0 is still required: the "Embed rsync" phase in `project.yml` harvests the binary + dylibs (incl. `libxxhash.0.dylib`) into `Resources/rsync/`.
- **Fan-out engine**: `FanOutCopier` (Swift actor) handles N-sources → M-destinations in one pass. One `BoundedChannel<Chunk>` per destination; source is read once and broadcast. Paranoid verify re-reads both source and dest from disk with `F_NOCACHE`. See `docs/architecture.md`.
- **Entry point**: `FilmCanApp.swift` — creates `MainView` window and `SettingsView`. No storyboards.
- **No CI, no linter, no formatter, no pre-commit hooks** — bare Xcode project.
- **Automated tests**: `FilmCan/Tests/` (~100 tests, real temp-dir disk I/O, no mocks) — `FanOutCopierIntegrationTests` (progress monotonicity, per-dest resume, sound routing, cancel/partial), `FanOutCopierSafetyTests` (cancel/unwritable), `ASCMHLWriter/Reader/Chain/Conformance` (validated against the reference `ascmhl` CLI), `C4HashTests`, `OrganizationTemplateTokenTests`.
- **State pattern**: `@StateObject` in views, `@Published` in ObservableObjects. `TransferViewModel` is the single source of truth for backup runs.

## Fan-Out Engine — Key Details

- `FanOutCopier` is a Swift **actor**; all mutable state (`completedFilesByDest`, `verifiedFilesByDest`, `verifiedBytesByDest`) is actor-isolated.
- `DestWriterResult.writtenFilePath` carries the **exact written path** (accounting for organization presets). Use this — never reconstruct the path from `destPath + rootName`.
- **Verify bar monotonicity**: each writer task snapshots `verifiedAtStart = await verifiedBytesForDest(dest)` before copy starts. Copy-phase progress emits carry this value so the verify bar never resets to 0% during the next file's copy.
- **Disk space pre-flight**: `FanOutCopier.run()` checks `DriveUtilities.liveAvailableBytes` against each dest's **needed** bytes (post-resume subset) before touching any file. Throws `Error.insufficientSpace` with a user-friendly message.
- **Paranoid verify on F_FULLFSYNC drives**: 1s settle delay before re-read to prevent false hash mismatches on drives that don't honor `F_FULLFSYNC` (exFAT USB, some SD cards).
- **Fan-out result explosion**: `TransferViewModel.explodeFanOutResult` converts one aggregate `TransferResult{destinationResults:[N]}` into N per-dest records so history and notifications show correct ✓/✗ per destination.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [qtld88/FilmCan](https://github.com/qtld88/FilmCan) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
