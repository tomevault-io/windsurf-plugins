---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Applite is a native macOS GUI application for Homebrew Casks, designed as an "app store for third-party apps" rather than a full Homebrew wrapper. Target audience is non-technical users who want simple app installation/management.

- **Language**: Swift with SwiftUI (`@Observable`, `@MainActor`, async/await)
- **Platform**: macOS 14+ (Apple Silicon and Intel)
- **Build System**: Xcode with Swift Package Manager for dependencies. Uses Xcode 16+ **file-system synchronized groups** (folder-based project): the on-disk folder structure *is* the project structure. Adding, removing, moving, or renaming source files needs **no `project.pbxproj` edits** — just change the files on disk and Xcode picks them up automatically. Do not hand-edit the pbxproj for file management.
- **Database**: GRDB.swift (SQLite) at `~/Library/Application Support/Applite/casks.sqlite`

## Build & Run

Open `Applite.xcodeproj` in Xcode and build/run (⌘R). Dependencies resolve automatically via SPM.

## Architecture

### Data Flow

1. **App Launch**: `ContentView.task(id: bootstrap.attempt)` calls `caskManager.bootstrapAndLoad()`
2. **First Run**: `HomebrewBootstrap` installs/validates brew; while `bootstrap.needsSetupOverlay` is true, `ContentView` covers the window with the non-dismissable `ComponentsInstallView` modal (`Features/Bootstrap/`). There is no separate onboarding flow — the app opens straight into the main UI
3. **Main UI**: `ContentView` with `NavigationSplitView` sidebar navigation
4. **Data Loading**: `CaskManager.loadData()` runs in two stages — catalog (DB-only, instant) then brew CLI state (slow). The UI lights up after stage 1; installed/outdated state arrives reactively as stage 2 completes.

### Key Components

The shared cask/brew engine lives under `Applite/Core/`, split into focused subfolders:

**Persistence** (`Applite/Core/Database/`)
- `AppDatabase` - Schema migrations, DatabasePool with WAL mode, FTS5 virtual table on `casks`
- `CaskRecord` - GRDB `FetchableRecord`/`PersistableRecord`, decodes from `CaskDTO`
- `CaskDatabaseService` - CRUD, FTS5 search (async), API sync

**Cask engine** (`Applite/Core/CaskCore/`) — the `@Observable` runtime layer
- `CaskViewModel` - `@Observable @MainActor` view model wrapping `CaskRecord` with runtime state (`isInstalled`, `isOutdated`, `progressState`)
- `CaskViewModelRegistry` - Single-identity store; `viewModels(for:)` is get-or-create so the same cask shares one VM across views. **Identity is `fullToken` everywhere** — DB primary key, registry key, brew ops. The bare `token` is not unique (two taps can each ship a "firefox"), so it must never key anything; it's indexed for lookup only
- `CaskDataLoader` - Orchestrates: `loadCatalogData()` (DB-only), `refreshInstalled()`/`refreshOutdated()` (brew CLI), `search(query:)` (FTS5). Defines `CategoryLoadResult` and `TapLoadResult`.
- `CaskWarning` - Warning enum (deprecated/disabled/caveat)
- `CaskProgressState`, `CaskLoadError` - install progress + load error types

**Plain models** (`Applite/Core/Models/`)
- `CaskDTO`, `CaskAdditionalInfo`, `BrewAnalytics` - decode-only DTOs for the Homebrew API/JSON
- `Category`, `CategoryLoadResult+LocalizedName`, `TapLoadResult`, `SidebarItem`, `SortingOptions`

Other `Core/` subfolders: `Core/Brew/` (brew CLI services + `BrewPaths`, `Shell`, `Installation/`), `Core/Preferences/`, `Core/Infrastructure/` (`AlertManager`, `AppPaths`, `SendNotification`, `MirrorEnvironment`, `NetworkProxyManager`, …).

**CaskManager** (`Applite/Core/CaskCore/CaskManager.swift`)
- Thin `@Observable @MainActor` coordinator owning `dataLoader`, `registry`, `brewService`
- `categories: [CategoryLoadResult]` and `taps: [TapLoadResult]` populated after stage 1
- `isResolvingInstalledState: Bool` is true during stage 2 (brew CLI); `isRefreshingCatalog: Bool` is true during a `forceSync` reload
- "Is brew usable" has exactly one owner: `bootstrap.phase` (`HomebrewBootstrap.Phase`). `isBrewReady` / `needsSetupOverlay` derive from it, and a broken brew surfaces only as the setup overlay. Don't add a parallel flag — `CaskManager.hasBrokenInstall` and `BrokenInstallView` were removed for exactly that reason. A `BrewService` op that finds the path invalid calls `recoverBrew` (wired to `bootstrap.run()`) instead of reacting on its own
- `alert: AlertManager` is the **main window's one alert surface** — brew failures, catalog/load failures and view-raised errors all queue in it, and `ContentView` presents it once at the window root via `.alertManager(_:)`. Rule: **one manager per window, bound at that window's root**; never inside a repeated view (binding it per cask card was the F5/P3-5 bug). Alerts carry their own buttons (`AppAlert.Action`), so nothing hand-rolls `.alert`. Windows that can't see that root (Settings' `UninstallView`) own a local one
- `loadData(forceSync:)` is non-throwing — it path-validates, runs stage 1, then stage 2, surfacing any failure through `alert` (with Retry/Quit actions). The same entry point powers initial load, the ⌘R menu action, and the "Refresh Catalog" prompt in Settings

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [milanvarady/Applite](https://github.com/milanvarady/Applite) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-06 -->
