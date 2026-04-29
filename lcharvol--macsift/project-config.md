---
trigger: always_on
description: Transparent macOS disk cleaning utility. SwiftUI, MVVM, macOS 26 (Tahoe), built with SwiftPM (no Xcode project file).
---

# MacSift — Working Notes for Claude

Transparent macOS disk cleaning utility. SwiftUI, MVVM, macOS 26 (Tahoe), built with SwiftPM (no Xcode project file).

## Build / run / test

```bash
# Compile only (fast, for type-checking)
swift build

# Run the full test suite
swift test

# Run a single suite
swift test --filter CleaningEngineIntegrationTests

# Build the .app bundle and launch (this is how the user runs it)
./build-app.sh
open MacSift.app
```

`build-app.sh` produces a proper `.app` bundle (with `Info.plist` + `AppIcon.icns`) from the SPM build output, ad-hoc signs it for stable TCC identity, and prints next steps. The bundle path is `MacSift.app/` at the repo root and is gitignored.

The user is on macOS 26.2 / Xcode 26.3. Deployment target is macOS 26.0.

## Project layout

```
MacSift/
├── App/                   # MacSiftApp entry point + AppState (@Published mode, dryRun, threshold)
├── Models/                # FileCategory, ScannedFile, ScanResult, FileGroup — pure value types, Sendable
├── Services/              # DiskScanner, CategoryClassifier, FileGrouper, CleaningEngine,
│                            TimeMachineService, ExclusionManager
├── ViewModels/            # ScanViewModel, CleaningViewModel — @MainActor, @Published, orchestrate services
├── Views/                 # SwiftUI views — WelcomeView, ScanProgressView, FileListSection,
│                            FileGroupRow, InspectorView, MainView, …
└── Utilities/             # FileSize+Formatting, FileDescriptions, BundleNames, Permissions
```

Keep the structure flat. Don't introduce package boundaries or sub-frameworks.

## File grouping architecture

`ScannedFile` is one underlying OS file. `FileGroup` is a display-layer
aggregation — multiple ScannedFiles that share a common owner (e.g., all files
under `~/Library/Caches/com.apple.Safari/` collapse into one "Safari" row).

- **`FileGrouper`** (Services) turns `[ScannedFile]` into `[FileGroup]` using
  per-category rules:
    - `.cache` / `.logs` / `.appData`: group by the first path component after
      `Library/<root>/` (the bundle id / folder name).
    - `.iosBackups`: group by the backup root folder under `MobileSync/Backup/`.
    - `.timeMachineSnapshots`: already 1:1 (each snapshot is one synthetic file).
    - `.tempFiles` / `.largeFiles`: kept as singleton groups.
- **Selection is still file-level** (`CleaningViewModel.selectedIDs: Set<String>`).
  Toggling a group selects/deselects all its underlying file ids at once via
  `toggleGroup`. A group is "fully selected" when all its file ids are in the
  selection set, "partially selected" when some are.
- **`FileGroup.topFiles`** is pre-computed at scan time (top N by size, via a
  partial sort). The inspector reads it directly — never sort the full
  `group.files` array in a view body.
- **`BundleNames.humanLabel(for:)`** translates `com.apple.Safari` → "Safari".
  Known apps are matched first; unknown reverse-DNS ids fall through to a
  heuristic that picks the most meaningful segment.
- **Rendering**: `FileListSection` renders `FileGroupRow`s, not individual files.
  Singleton groups still appear as one-row entries.

## Conventions

- **MVVM, strict.** Views render state and call methods; never call services directly.
- **Models are `struct` + `Sendable`.** No reference types in the data flow.
- **Services are `struct` or `enum`** unless they genuinely need identity (none currently do — `DiskScanner` was an actor and got demoted).
- **ViewModels are `@MainActor final class`** with `@Published` properties. Heavy work hops off main via `Task.detached`.
- **No emojis in code.** Sparingly in commit messages and only when it adds info.
- **No `design: .rounded` fonts.** We removed all of them — system semibold is the house style.
- **No gradients.** Use `.tint`, `.accentColor`, system materials. Liquid Glass via `.glassEffect()` and `.buttonStyle(.glass)` / `.buttonStyle(.glassProminent)`.
- **Use `.monospacedDigit()` on every number** that updates frequently (sizes, counters) to avoid jitter.

## Performance lessons (read before touching the file list)

We've already paid for these. Don't undo them without a good reason.

1. **Pre-sort once at end of scan, cache in `ScanViewModel.sortedFilesByCategory` and `allSortedFiles`.** Never sort in a `View.body`.
2. **Selection is `Set<String>` (file id), not `Set<ScannedFile>`.** The id is a stable SHA-256 of the URL path, derived in `ScannedFile.init`. Stable across re-scans.
3. **`FileDetailView` is `Equatable`** and used with `.equatable()`. The `==` ignores closures and only compares `file.id`, `isSelected`, `isAdvanced`. SwiftUI skips re-rendering rows whose props didn't change.
4. **`FileListSection` is a separate view** that takes plain values (not VMs). This isolates its body from unrelated parent re-renders.
5. **Default file list cap is 300 rows** (sorted by size). The `Show all` button raises the cap.
6. **Use `List` not `LazyVStack`.** On macOS, `List` is backed by `NSTableView` which is far faster for thousands of rows.
7. **NEVER add `.contextMenu` to row views.** Closure capture per row is expensive at scale. Put per-file actions in a detail panel or a hover-revealed menu, not on every row.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Lcharvol/MacSift](https://github.com/Lcharvol/MacSift) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-19 -->
