---
trigger: always_on
description: Brewy is a native macOS GUI for managing Homebrew packages, written in Swift/SwiftUI. It lets users browse, search, install, upgrade, pin, and uninstall formulae and casks without opening Terminal. The project is open source (AGPL-3.0-only) and lives at https://github.com/starhaven-io/Brewy.
---

# Brewy — Claude Project Context

Brewy is a native macOS GUI for managing Homebrew packages, written in Swift/SwiftUI. It lets users browse, search, install, upgrade, pin, and uninstall formulae and casks without opening Terminal. The project is open source (AGPL-3.0-only) and lives at https://github.com/starhaven-io/Brewy.

## Project overview

- **Platform:** macOS 15.0+ (Apple Silicon), built with Xcode 16+
- **Language:** Swift, SwiftUI (100%)
- **Architecture:** MVVM using `@Observable` and SwiftUI Environment injection
- **Only external dependency:** Sparkle via Swift Package Manager for auto-updates
- **Bundle ID:** `io.linnane.brewy`
- **License:** AGPL-3.0-only

## Repository structure

```
Brewy/
├── Brewy/
│   ├── BrewyApp.swift                      # @main entry, WindowGroup, MenuBarExtra, Sparkle updater
│   ├── Info.plist                           # Sparkle feed URL + EdDSA public key
│   ├── Assets.xcassets/                     # App icon, accent color
│   ├── Models/
│   │   ├── BrewService.swift               # @Observable service: core state, caching, derived state, CLI orchestration
│   │   ├── BrewService+Fetching.swift      # Installed package and outdated package fetching
│   │   ├── BrewService+Actions.swift       # Package action helpers (install, uninstall, upgrade, pin, etc.)
│   │   ├── BrewService+PackageDetail.swift # Single-package detail fetching
│   │   ├── BrewService+History.swift       # Action history recording, persistence, retry, outdated merge helper
│   │   ├── BrewService+Groups.swift        # User-created package group CRUD and persistence
│   │   ├── BrewService+DryRun.swift        # Dry-run previews for autoremove and cleanup
│   │   ├── BrewJSONTypes.swift             # Brew JSON v2 Codable response types (extracted from PackageModel)
│   │   ├── PackageModel.swift              # Data models: BrewPackage, BrewTap, SidebarCategory, PackageGroup, ActionHistoryEntry, BrewServiceItem, BrewConfig, appcast parser
│   │   ├── CommandRunner.swift             # Process execution with timeout, cancellation, thread-safe pipe reading, CommandRunning protocol
│   │   ├── MasService.swift                # Mac App Store integration via mas CLI (list, outdated, install)
│   │   ├── ServicesService.swift           # Homebrew services management (fetch, start, stop, restart, cleanup)
│   │   └── TapHealthChecker.swift          # Async GitHub API tap health detection (archived, moved, missing)
│   └── Views/
│       ├── ContentView.swift               # NavigationSplitView (3-column), toolbar, state management
│       ├── SidebarView.swift               # Category list (13 categories, see SidebarCategory enum)
│       ├── PackageListView.swift           # Package list with search, selection toggles for bulk upgrade
│       ├── PackageDetailView.swift         # Detail pane: info, dependencies, reverse deps, actions
│       ├── DiscoverView.swift              # Search all of Homebrew (formulae + casks)
│       ├── GroupsView.swift                # Custom package groups with CRUD UI
│       ├── HistoryView.swift               # Action history with review and retry
│       ├── ServicesView.swift              # Homebrew services control UI
│       ├── MasSetupView.swift              # Mac App Store setup/configuration UI
│       ├── MaintenanceView.swift           # brew doctor, cleanup, autoremove, cache management
│       ├── DryRunConfirmationSheet.swift   # Preview modal for cleanup/autoremove operations
│       ├── SharedViews.swift               # Reusable components: FlowLayout, ConsoleOutput, ActionOverlay
│       ├── SettingsView.swift              # Brew path, auto-refresh interval, theme
│       ├── TapListView.swift               # Add/remove taps
│       └── WhatsNewView.swift              # Release notes from Sparkle appcast
├── BrewyTests/
│   ├── TestHelpers.swift                   # Shared test utilities (MockCommandRunner, etc.)
│   ├── BrewServiceTests.swift              # Core service logic: derived state, reverse deps, leaves, category routing
│   ├── BrewServiceAsyncTests.swift         # Async tests: refresh, search, bulk upgrade, actions
│   ├── BrewServiceDetailTests.swift        # Maintenance, dry-run, info caching, tap management, retry, error handling
│   ├── CommandRunnerTests.swift            # CommandRunner process execution + MockCommandRunner behavior
│   ├── PackageModelTests.swift             # Package model JSON parsing, equality, hashing
│   ├── TapAndConfigTests.swift             # Tap health, GitHub URL parsing, appcast parsing, config parsing
│   ├── GroupsAndMasTests.swift             # Package groups, Mac App Store parsing
│   ├── HistoryTests.swift                  # Action history recording and retry
│   ├── ServicesTests.swift                 # Services parsing and integration
│   └── JSONAndServicesTests.swift          # JSON parsing edge cases and services
├── BrewyUITests/
│   └── SidebarNavigationUITests.swift      # UI tests for sidebar navigation
├── Brewy.xcodeproj/
├── .github/
│   ├── workflows/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [starhaven-io/Brewy](https://github.com/starhaven-io/Brewy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
