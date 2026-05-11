---
trigger: always_on
description: Native macOS menu bar system monitor. Inspired by iStat Menus. Personal use, not App Store.
---

# MacStats — Agent Guide

Native macOS menu bar system monitor. Inspired by iStat Menus. Personal use, not App Store.

## Product context

- Replace macOS Activity Monitor for quick glances at CPU/RAM/Disk/Network/Battery.
- Menu bar first: compact metrics always visible; dropdown for detail.
- Per-process tops (CPU/RAM/Disk) so the user can spot hogs without opening Activity Monitor.
- Target audience: the repo owner. No localization, no onboarding, no telemetry.

### Non-goals

- App Store distribution (no sandboxing, no notarization pipeline).
- Cross-platform. macOS 13+ only (uses modern SwiftUI APIs + IOKit).
- Fan RPM and SMC voltage/current sensors. Private SMC keys vary per chip — deferred until needed. (CPU/GPU/SOC temperatures *are* supported via IOHID; see below.)

## Stack

- Swift 6.3 toolchain + Swift 6 **strict concurrency** language mode (`.swiftLanguageMode(.v6)` in `Package.swift`). Every `Sample` struct is `Sendable`.
- SwiftUI + AppKit (`NSStatusItem` for menu bar, SwiftUI for content).
- Swift Package Manager (no Xcode project). `Package.swift` is the source of truth.
- macOS 13+ deployment target.
- No external dependencies.

## Build / run

```bash
./Scripts/run.sh           # debug bundle + launch
./Scripts/bundle.sh release  # release .app
swift build -c debug       # compile only
pkill -x MacStats          # kill running instance
```

`run.sh` wraps the SPM binary into a proper `.app` with `Info.plist` (sets `LSUIElement=true` so the app has no Dock icon). `bundle.sh` also copies `Resources/AppIcon.icns` into the bundle and wires `CFBundleIconFile = AppIcon` so Finder / About show the MacStats icon.

When iterating, kill before rebuild (`pkill -x MacStats`). If a change looks like it didn't apply, run `swift package clean` — SPM has occasionally served stale binaries in this repo.

## Architecture

```
Sources/MacStats/
├── MacStatsApp.swift            # @main, AppDelegate; spawns StatusBarController + MainWindowController; pkill orphan nettops on launch/quit
├── StatusBarController.swift    # owns N NSStatusItems (one per metric) + shared NSPopover; retains detail + nettop sampling while popover is shown
├── MainWindowController.swift   # NSWindowController hosting MainWindowView (sidebar + detail panes)
├── SystemStats.swift            # @MainActor ObservableObject + actor StatsSampler; refcounted detail/full-process/nettop tiers; ThermalLevel enum
├── DisplayPreferences.swift     # BarMetric enum + which metrics show in menu bar (UserDefaults-backed)
├── MenuBarSnapshot.swift        # frozen copy of selected metrics for the status bar
├── Formatters.swift             # byte/rate/percent formatting
├── ProcessKill.swift            # confirm-and-kill helper used by leader rows
├── Monitors/                    # stateless-ish samplers, one per hardware domain
│   ├── CPUMonitor.swift         # host_statistics HOST_CPU_LOAD_INFO
│   ├── MemoryMonitor.swift      # host_statistics64 HOST_VM_INFO64
│   ├── NetworkMonitor.swift     # getifaddrs + if_data
│   ├── DiskMonitor.swift        # IOKit IOBlockStorageDriver + cached volume stats
│   ├── BatteryMonitor.swift     # IOPowerSources
│   ├── ProcessMonitor.swift     # libproc: proc_listpids + PROC_PIDTASKALLINFO + rusage
│   ├── NetworkProcessMonitor.swift  # spawns `nettop` and parses per-process bytes_in/out
│   ├── TemperatureMonitor.swift # private IOHIDEventSystemClient: CPU/GPU/SOC thermal sensors
│   └── SamplingMath.swift       # shared delta / rate helpers (handles counter rollover)
└── Views/
    ├── SingleMetricLabel.swift     # one metric in the menu bar (icon above compact value)
    ├── MenuBarContentView.swift    # dropdown / popover content (header + sections + prefs + quit)
    ├── MenuBarPrefsView.swift      # 3-col grid of checkboxes for which metrics show in bar
    ├── TopProcessesView.swift      # tabbed top processes (CPU/RAM/Disk/Network/Energy)
    ├── MainWindowView.swift        # sidebar nav (Overview / Hardware / Activity)
    ├── PaneKit.swift               # shared pane primitives: PaneHeader, MetricCard, AreaSpark, DualAreaSpark, ScaleHelper
    └── Panes/
        ├── DashboardPane.swift     # at-a-glance card grid (cpu, mem, network, disk, battery, temperature)
        ├── MetricPanes.swift       # CPU / Memory / Disk / Network / Battery / Temperature panes + LeaderList
        └── ProcessesPane.swift     # full filterable / sortable process table

Resources/
└── AppIcon.icns                # built via iconutil from design_handoff_macstats_logo/

design_handoff_macstats_logo/   # canonical icon source (SVG + sized PNGs + README)
```

### Data flow

1. `SystemStats` (MainActor `ObservableObject`) holds one `@Published snapshot: SystemSnapshot`. `SystemSnapshot` is a `Sendable` value type bundling every monitor's latest `Sample` + the CPU history array + pre-sorted `ProcessLeaders` (top N per CPU/RAM/Disk).
2. Sampling runs in a **separate `actor StatsSampler`**, off the main thread. `SystemStats.init` starts a `Task` loop that calls `await sampler.sample(...)` every 1s and assigns the returned snapshot to `snapshot`. Views observe `snapshot` via `@Published` and see atomic, consistent updates.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [luizhcastro/mac-stats](https://github.com/luizhcastro/mac-stats) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
