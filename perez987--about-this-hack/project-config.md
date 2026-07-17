---
trigger: always_on
description: **About This Hack** is a macOS app that recreates the classic pre-Ventura "About This Mac" window with modern enhancements. It targets both Hackintosh users (PCs running macOS via OpenCore or Clover bootloaders) and real Mac users. It displays hardware info (CPU, RAM, GPU, displays, storage, audio codec), bootloader version, serial number, and support links.
---

# About This Hack – Codebase Guide for Claude

## Project Purpose

**About This Hack** is a macOS app that recreates the classic pre-Ventura "About This Mac" window with modern enhancements. It targets both Hackintosh users (PCs running macOS via OpenCore or Clover bootloaders) and real Mac users. It displays hardware info (CPU, RAM, GPU, displays, storage, audio codec), bootloader version, serial number, and support links.

- macOS 13.5 (Ventura) minimum deployment target
- Pure SwiftUI UI (migrated from AppKit/XIB)
- Storyboard-free (AppDelegate bootstraps everything manually)
- Auto-updater via Sparkle framework (SPM dependency)
- Localized into: English (`en`), Spanish (`es`), French (`fr`), Italian (`it`)
<!-- - Transparent/liquid-glass window style (`.ultraThinMaterial` backgrounds) -->

## Tech Stack

| Area | Technology |
|---|---|
| Language | Swift 5 (+ one Objective-C file for SIP reading) |
| UI framework | SwiftUI (macOS 13.5+) |
| App lifecycle | AppKit (`NSApplicationDelegate`, no storyboard) |
| Auto-updater | Sparkle via SPM package (`UpdaterController` wrapping `SPUStandardUpdaterController`) |
| Shell execution | Custom `run(_ command: String) -> String` via `/bin/zsh` subprocess |
| Localization | `NSLocalizedString` + `.strings` files per language |
| Persistence | `UserDefaults.standard` (window frame, custom logo path, language) |
| IPC / system data | `system_profiler`, `diskutil`, `nvram`, `sw_vers`, `sysctlbyname`, IOKit, `/bin/zsh` subprocesses |

## Repository Layout

```
About-This-Hack/
├── CLAUDE.md                          ← this file
├── README.md
├── DOCS/                              ← Developer documentation (markdown)
│   ├── AppKit-XIB-to-SwiftUI.md
│   ├── ATHLogger-to-print.md
│   ├── App-damaged.md
│   ├── Console-warning.md
│   ├── Replace-custom-logo.md
│   ├── Transparent-windows.md
│   └── appcast-backup.xml
├── Images/                            ← README screenshots (not bundled)
├── Releases-changelog.md
├── appcast.xml
├── appcast-glass.xml
└── About This Hack/                   ← All source code
    ├── AppDelegate.swift              ← App entry point, window/menu setup
    ├── HardwareCollector.swift        ← Central data collector (singleton)
    ├── About_This_Hack.entitlements
    ├── Info.plist
    ├── Assets.xcassets/               ← Images (OS badges, disk icons, display icons)
    ├── HardwareCollectors/            ← Individual hardware data collectors
    │   ├── HCAudio.swift
    │   ├── HCBootloader.swift
    │   ├── HCCPU.swift
    │   ├── HCDisplay.swift
    │   ├── HCGPU.swift
    │   ├── HCMacModel.swift
    │   ├── HCRAM.swift
    │   ├── HCSerialNumber.swift
    │   ├── HCStartupDisk.swift
    │   └── HCVersion.swift
    ├── Models/                        ← Shared state, data init, utilities
    │   ├── CreateDataFiles.swift      ← Async creation of temp data files
    │   ├── InitGlobalVariables.swift  ← All global constants and paths
    │   ├── LoadingIndicatorController.swift
    │   ├── SystemFunctions.swift      ← getSysctlValueByKey(), IOKit port helpers, Bundle ext.
    │   ├── Tooltips.swift             ← Lazy tooltip strings (expensive to compute)
    │   ├── UpdateController.swift     ← Sparkle updater controller wrapper
    ├── Utilities/                     ← Small helpers
    │   ├── CustomLogoConstants.swift  ← UserDefaults key + Notification.Name
    │   ├── InsertExtension.swift      ← String/Data extensions
    │   ├── Reachability.swift         ← Network reachability helper
    │   └── Shell.swift                ← run() function definition
    ├── Views/                         ← All SwiftUI views + AppState
    │   ├── ContentView.swift          ← Root view: LoadingView or FakeSidebarLayout
    │   ├── ViewController.swift       ← OverviewView + OverviewViewModel
    │   ├── ViewControllerDisplays.swift ← DisplaysView + DisplaysViewModel
    │   ├── ViewControllerStorage.swift  ← StorageView + StorageViewModel
    │   ├── ViewControllerAudio.swift    ← AudioView + AudioViewModel
    │   ├── ViewControllerSupport.swift  ← SupportView
    │   ├── SettingsView.swift           ← Settings window (custom logo)
    │   ├── SettingsWindowController.swift
    │   ├── LanguageSelectorView.swift
    │   ├── LanguageSelectorWindowController.swift
    │   └── WindowController.swift       ← AppState: ObservableObject shared state
    └── {en,es,fr,it,de}.lproj/
        └── Localizable.strings
```

## Architecture & Data Flow

### Startup sequence

```
AppDelegate.init()
  ├── AppState.shared: registers DataFilesCreated notification observer
  │     and checks CreateDataFiles.dataFilesCreated in case files are already ready
  └── CreateDataFiles.getInitDataFilesAsync { }
        └── background thread: runs system_profiler / diskutil commands
              → writes temp files to InitGlobVar.athDirectory (`FileManager.default.temporaryDirectory/.ath`)
              → posts DataFilesCreatedNotification


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [perez987/About-This-Hack](https://github.com/perez987/About-This-Hack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-17 -->
