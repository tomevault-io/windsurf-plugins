---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
swift build                          # debug build
swift test                           # full suite (swift-testing, ~260 tests, sub-second after build)
swift test --filter "TreeActions"    # one suite; also --filter "SuiteName/testFunctionName"
./scripts/package-release.sh         # release .app bundle + zip → dist/ (version from DirWiz/Info.plist)
.build/debug/dirwiz-cli scan <path> [--json] [--min-size N] [--max-depth N] [-q]
.build/debug/dirwiz-cli duplicates|info|benchmark|snapshot|diff <path>
```

CI (`.github/workflows/ci.yml`) runs `swift build` + `swift test` on `macos-15` for pushes to master and PRs.

Release script notes: signs with the first local Apple identity found, else ad-hoc (`DIRWIZ_CODESIGN_IDENTITY` overrides; `DIRWIZ_DIST_DIR` overrides output dir). Grant Full Disk Access to the installed `/Applications/DirWiz.app`, never to `.build` binaries - ad-hoc rebuilds can lose FDA because macOS ties privacy grants to code identity.

## Targets and layering

SwiftPM, swift-tools 6.0, **all targets pinned to Swift 5 language mode** (the test target is not pinned and compiles under Swift 6). macOS 15+. Zero external dependencies - keep it that way.

- `Sources/DirWizCore` - scanner, file tree, duplicate/hardlink detection, analyzers, temporal diff, CLI argument parsing. No UI imports.
- `Sources/DirWizUI` - `AppState` + SwiftUI views + Metal cushion treemap.
- `DirWiz/` (app) and `CLI/` (dirwiz-cli) are executable targets. **The test target can only import DirWizCore and DirWizUI** - logic that needs tests must live in DirWizCore, not the executables (pattern: `CLIArguments.swift`, `TemporalDiffSummary.swift`).

## The file tree - core invariants

`FileTree` (Sources/DirWizCore/Scanner/FileNode.swift) is a flat array of packed `FileNode` structs plus a shared string pool. Everything depends on these rules:

- Nodes reference each other by index (`parentIndex`, `firstChildIndex` + `childCount` = one contiguous child slice). Parent index < child index for any real scan.
- Nodes are appended, never removed - **except `removeSubtree`, which compacts the array and renumbers every index**. Any index held across that call is garbage. That is why:
  - `TreeActions.batchTrash(paths:tree:)` re-resolves each path against the current tree immediately before trashing it; the index-based `batchTrash(nodeIndices:)` is only safe for a single index.
  - After any mutating action, `AppState.invalidateAfterTreeMutation()` (AppState+Analysis.swift) is the single reset point: it clears all index-keyed state (search results, recency factors, temporal-diff arrays) and bumps the treemap layout revision via `scanProgress.publishCounters(forceLayoutRevision: true)`. The treemap only relayouts on tree identity / root index / revision changes - forget the bump and it silently renders stale rects with wrong node indices.
- `FileTree` is `@unchecked Sendable` guarded by an internal mutex. Read through `node(at:)`, `nodesSnapshot()`, `pathBuildingSnapshot()` - do not read `.nodes` directly from concurrent contexts.
- Post-scan analyzers walk snapshots via `FileTree.forEachFileInSnapshot` (single blessed walk with a uniform cancellation cadence) and locate nodes by path components via `FileTree.descendPath`. Do not hand-roll a new whole-tree loop in an analyzer.

## Scanner

`getattrlistbulk` raw-buffer parsing with unsafe pointers (`FilesystemProvider.swift`, hot paths in `FileScanner.swift`) - the parser uses FIXED offsets that depend on `FSOPT_PACK_INVAL_ATTRS` packing every requested attribute, so adding/removing a requested attr shifts every later offset (see the offset constants + parsing contract tests). Symlinks are intentionally skipped. Scans also capture `ATTR_FILE_LINKCOUNT` into the `hasMultipleHardlinks` node flag (`FileTree.linkCountsCaptured` marks the flags trustworthy); hardlink groups auto-populate post-scan/post-mutation via `AppState.refreshHardlinkGroups()` - the Hardlinks tab has no run button. The app path defers bundle sizing to a background pass so first paint is fast; the CLI sizes bundles inline. Env knobs: `DIRWIZ_SCAN_WORKERS`, `DIRWIZ_DEFER_TREE`, `DIRWIZ_SKIP_BUNDLE_SIZES`, `DIRWIZ_BUNDLE_WORKERS`, `DIRWIZ_BULK_BUFFER_BYTES`, `DIRWIZ_NO_WARM_START`. Perf landmine: never `reserveCapacity(count + smallDelta)` on a per-directory/per-item path - it defeats Array's amortized growth and turns scanning into O(n²) (masked below the 500k init reservation, brutal above it; use `reserveNodeCapacity`/geometric growth). Scan-time treemap layouts are deliberately sparse + depth-capped (`ScanTimeLayoutBudget`) so live building doesn't starve the scanner.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [okturan/dirwiz](https://github.com/okturan/dirwiz) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-30 -->
