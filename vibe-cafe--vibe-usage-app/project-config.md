---
trigger: always_on
description: AI agent guidance for the vibe-usage-app repository.
---

# AGENTS.md

AI agent guidance for the vibe-usage-app repository.

## Repository Map

```
vibe-usage-app/                    # SwiftUI macOS menu bar app (SPM, Swift 6, macOS 14+)
├── Package.swift                  # SPM manifest (Sparkle dependency)
├── VibeUsage/
│   ├── Info.plist                 # Bundle metadata (versions, Sparkle SUFeedURL, SUPublicEDKey)
│   ├── App/
│   │   ├── VibeUsageApp.swift     # @main entry, MenuBarExtra scene
│   │   └── AppResources.swift     # Bundle.appResources helper
│   ├── Models/
│   │   ├── AppState.swift         # @Observable central state (buckets, filters, timeRange, sync)
│   │   ├── AppConfig.swift        # Version string, API URL, debug/release config
│   │   ├── UsageBucket.swift      # Codable data model (source, model, project, hostname, tokens, cost)
│   │   └── Config.swift           # Persistent config (apiKey, apiUrl) in ~/.vibe-usage/
│   ├── Views/
│   │   ├── PopoverView.swift      # Main dashboard container (520px wide popover)
│   │   ├── SummaryCardsView.swift # 5 stat cards (cost, total tokens, cached tokens, active duration, total duration)
│   │   ├── BarChartView.swift     # Custom-drawn bar chart (hourly/daily trend)
│   │   ├── DistributionChartsView.swift  # 4 donut pie charts (terminal, tool, model, project)
│   │   ├── FilterTagsView.swift   # Filter pills for source/model/project/hostname
│   │   └── SettingsView.swift     # Settings form (API key, menu bar prefs, auto-start, updates)
│   ├── Services/
│   │   ├── APIClient.swift        # HTTP client for /api/usage (Bearer auth with vbu_ key)
│   │   ├── SyncEngine.swift       # Orchestrates CLI sync (runs @vibe-cafe/vibe-usage via Node/Bun)
│   │   ├── SyncScheduler.swift    # 30-minute interval auto-sync timer
│   │   ├── CLIBridge.swift        # Executes vibe-usage CLI as subprocess
│   │   ├── RuntimeDetector.swift  # Finds Node.js or Bun runtime on the system
│   │   ├── UpdaterViewModel.swift # Sparkle SPUUpdater bridge + SPUUpdaterDelegate proxy (publishes availableUpdate)
│   │   ├── MenuBarController.swift # NSStatusItem + custom borderless popover panel (multi-line title, animated open/close)
│   │   ├── PopoverPanel.swift     # NSPanel subclass that becomes key for TextField input
│   │   ├── SettingsWindowController.swift  # NSWindow wrapper (LSUIElement keyboard workaround)
│   │   └── ActivationCoordinator.swift     # Centralizes NSApp.activationPolicy across popup + Settings
│   ├── Utils/
│   │   ├── Formatters.swift       # Number, cost, date, time formatting
│   │   └── Log.swift              # Debug logging
│   └── Resources/
│       └── Assets.xcassets/       # App icon, menu bar icon
├── scripts/
│   ├── build-app.sh               # Build + sign + notarize pipeline (runs check-version.sh first)
│   ├── check-version.sh           # Guards AppConfig/Info.plist version sync + monotonic CFBundleVersion
│   └── generate-appcast.sh        # Generate Sparkle appcast.xml
└── dist/                          # Build output (gitignored)
    ├── Vibe Usage.app
    ├── VibeUsage.dmg
    ├── VibeUsage.zip
    └── appcast.xml
```

## Quick Commands

```bash
swift build                              # Debug build
swift build -c release                   # Release build
./scripts/check-version.sh               # Validate version sync across AppConfig + Info.plist
./scripts/build-app.sh                   # Build + codesign .app (runs check-version.sh first)
./scripts/build-app.sh --notarize        # Full pipeline: build + sign + notarize + DMG
./scripts/generate-appcast.sh            # Generate appcast.xml from dist/VibeUsage.zip
```

## Architecture

### App Type
LSUIElement menu bar app — no dock icon, no main window. `AppDelegate` owns a `MenuBarController` that manages an `NSStatusItem` plus a borderless `PopoverPanel` (custom NSPanel) hosting the SwiftUI dashboard. We dropped `MenuBarExtra` so the status item can render multi-line text via `NSHostingView` (cost over tokens) and the panel can use a custom open/close animation anchored to the icon.

### State Management
`AppState` is `@Observable` and injected via `@Environment`. All views read from it. No Combine, no ObservableObject (except `UpdaterViewModel` which bridges Sparkle's KVO).

### View Hierarchy
```
VibeUsageApp → AppDelegate → MenuBarController (NSStatusItem + PopoverPanel)
└── PopoverView (520px wide, hosted in NSHostingView pinned to panel.contentView)
    ├── unconfiguredView          # First-run API key setup
    └── dashboardView
        ├── headerBar             # Title, "查看详情" link, time range (1D/7D/30D), settings gear
        ├── ScrollView
        │   ├── FilterTagsView    # Source/model/project/hostname filter pills
        │   ├── SummaryCardsView  # 5 stat cards
        │   ├── BarChartView      # Trend chart (hourly or daily)
        │   └── DistributionChartsView  # 4 donut charts (2x2 grid)
        └── footerBar             # Sync status, refresh, quit
```

### Data Flow
1. `APIClient.fetchUsage(days:)` fetches from `/api/usage` with Bearer token auth
2. Response decoded into `[UsageBucket]`, stored in `AppState.buckets`
3. Views compute filtered data locally: `appState.buckets.filter { ... appState.filters ... }`
4. Charts aggregate filtered buckets by time key or dimension

### Sync Pipeline

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vibe-cafe/vibe-usage-app](https://github.com/vibe-cafe/vibe-usage-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
