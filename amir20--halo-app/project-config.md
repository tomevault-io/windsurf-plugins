---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

**Halo** — a native SwiftUI disk-space visualizer for macOS. It scans a directory tree, classifies what's using space, and renders an interactive donut with two lenses (**by folder** / **by type**) plus a synced breakdown sidebar. Built on **DiskKit**, a small library that does the parallel filesystem walk and builds a classified tree.

Pure SwiftPM package with exactly **one external dependency: Sparkle** (auto-update), linked into the Halo target only — DiskKit has none. Targets **macOS 26+ / Swift 6.2 (Xcode 26)** and builds in **Swift 6 language mode with strict data-race checking**.

## Commands

A `Makefile` is the front door — run `make` to list targets.

```sh
make build   # release build of the Halo binary (swift build -c release --product Halo)
make test    # swift test
make run     # build & launch the app from source
make app     # -> Halo.app  (release binary + Info.plist + icon + ad-hoc signature)
make dmg     # -> Halo.dmg  (drag-to-install; depends on `app`)
make icon    # regenerate Icons/AppIcon.icns (only if Icons/make-icon.swift changed)
make clean
```

Run a single test by `Suite/test` or by suite:

```sh
swift test --filter HaloTests.ScanModelTests/testDonutHoverHitsTheArcUnderTheCursor
swift test --filter DiskKitTests          # whole target
```

The app bundle is produced by the `bundle-app` **package plugin** (not a script): `make app` runs `swift package --disable-sandbox --allow-writing-to-package-directory bundle-app Halo`. The plugin builds release, writes a **binary** `Info.plist` via `PropertyListSerialization` (stamping `VERSION`, default `0.0.0`, plus Sparkle's `SUFeedURL`/`SUPublicEDKey`), copies `Icons/AppIcon.icns`, embeds `Sparkle.framework` into `Contents/Frameworks` (adding the `@executable_path/../Frameworks` rpath), **re-signs Sparkle's nested XPC services/Autoupdate/Updater.app deepest-first** (library validation under the hardened runtime requires nested code signed by our team), then signs the app. CI (`.github/workflows/ci.yml`, macos-26 runner) builds + tests and uploads `Halo.dmg` as an artifact on every push/PR.

**Auto-update (Sparkle).** Installed apps poll `SUFeedURL` = `https://github.com/amir20/Halo.app/releases/latest/download/appcast.xml`. The CI `release` job (on `v*` tags) EdDSA-signs the notarized DMG with `sign_update` (key from the `SPARKLE_PRIVATE_KEY` secret; tools ship inside the Sparkle SPM artifact under `.build/artifacts/sparkle/Sparkle/bin/`) and attaches a single-item `appcast.xml` to the release — `releases/latest/download/` keeps the feed current with no hosting. The updater only starts when running from a real `.app` bundle, so `swift run`/tests are unaffected.

> The GUI is a SwiftUI `App`, so it can't be smoke-tested headlessly here — hover/visual behavior must be verified by a human running the app. Logic that *can* be tested lives in `HaloTests` (the executable target is `@testable`-importable).

## Architecture

Two modules: **DiskKit** (scan engine + model, no UI) and **Halo** (SwiftUI, depends on DiskKit). The data flows scanner → `DirNode` tree → `ScanModel` (view-model) → views.

### Scanning (`DiskKit/TreeScanner.swift`)
Parallel walk over a single shared `NodeQueue` (LIFO stack + pending counter guarded by `NSCondition`) with N workers via `DispatchQueue.concurrentPerform`. **All directories share one queue** — this is deliberate, so a single huge subtree (`~/Library`, a giant `node_modules`) can't bottleneck the scan. Workers fill a mutable `BuildNode` tree (each node written by exactly one worker — the single-writer invariant behind its `@unchecked Sendable`), which is frozen into the immutable `DirNode` tree at the end. The walk counts hard-linked inodes once (like `du`), never crosses onto another device (mount points/firmlinks), and counts unreadable dirs in `ScanProgress.skipped` so the UI can disclose the blind spot.

`scanStreaming` is what the GUI uses: it reports the root with size-0 **placeholder** children immediately, then reports each top-level subtree via `onChild` the moment it finishes (tracked by `SubtreeTracker`'s per-subtree pending counts). A `ScanToken` cancels the walk cooperatively (it drains the queue; `onDone` still fires). `scan` is the blocking whole-tree variant (tests). Both produce byte-identical trees.

### The model (`Halo/ScanModel.swift`) — read this first
`@MainActor @Observable` view-model. **Critical invariant:** the scope-derived properties — `segments`, `arcs`, `reclTotal`, `reclaimTargets`, `expandedLocations` — are **stored**, rebuilt by `refresh()` only when the *scope* changes (`root` / `path` / `mode`, i.e. wherever `sweepKey` is bumped). They are **not** recomputed on `hover`/`expanded`. This is a performance fix: the `Derive` functions are O(subtree), and the views read these inside animated bodies that re-run on every hover/animation frame. **Do not turn them back into computed properties or recompute them on hover.** `focus` stays computed because it's a cheap lookup over cached `segments`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [amir20/Halo.app](https://github.com/amir20/Halo.app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
