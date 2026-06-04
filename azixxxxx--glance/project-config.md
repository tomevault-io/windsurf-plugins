---
trigger: always_on
description: Modern macOS status bar replacement with liquid glass UI, native Spaces support, and custom widgets.
---

# Glance — Custom macOS Status Bar

Modern macOS status bar replacement with liquid glass UI, native Spaces support, and custom widgets.

**Version:** 1.3.0
**Author:** azixxxxx (Azim Sukhanov)
**GitHub:** https://github.com/azixxxxx/glance
**Bundle ID:** `com.azimsukhanov.glance`

## System Context

- **Hardware:** Mac Mini M4, 1080p display
- **Network:** Wi-Fi only (no Ethernet)
- **Window Manager:** Native macOS (no yabai, no AeroSpace)
- **macOS:** Sequoia
- **Config file:** `~/.glance-config.toml`
- **Deployed at:** `/Applications/Glance.app`

## Build & Deploy

```bash
# Build (from project root)
xcodebuild -project Glance.xcodeproj -scheme Glance -configuration Release \
  -derivedDataPath build build \
  CODE_SIGN_IDENTITY=- CODE_SIGNING_REQUIRED=NO CODE_SIGNING_ALLOWED=NO

# Deploy (MUST kill, rm -rf, then copy — cp alone won't overwrite a running app)
pkill -x Glance; sleep 2
rm -rf /Applications/Glance.app
cp -R build/Build/Products/Release/Glance.app /Applications/Glance.app
open /Applications/Glance.app

# Build DMG + ZIP for distribution
./scripts/build-dmg.sh
```

**Important:** Always `rm -rf` before `cp -R`. Otherwise the old binary may persist.

**Style files must be moved before build** — Xcode auto-discovers all `.swift` in the tree:
```bash
mkdir -p /tmp/glance-styles-backup
mv Glance/Styles/{Glass,Minimal,Solid,System}Style.swift /tmp/glance-styles-backup/
# ... build ...
cp /tmp/glance-styles-backup/*.swift Glance/Styles/
```

## Project Structure

```
Glance/
├── Info.plist                          # LSUIElement=true, Sparkle keys, Location keys
├── AppDelegate.swift                   # App lifecycle, tray icon, login item, Sparkle updater
├── GlanceApp.swift                     # SwiftUI app entry (@main)
├── Constants.swift                     # App constants
├── Config/
│   ├── ConfigManager.swift             # Config loading, file watching, live reload
│   ├── ConfigModels.swift              # RootToml, Config, style support
│   ├── AppearanceConfig.swift          # Visual params struct + resolvedCornerRadius
│   ├── PresetRegistry.swift            # 11 built-in presets (Preset enum)
│   └── CustomPresetStore.swift         # User presets in ~/Library/Application Support/glance/presets/
├── MenuBarPopup/                       # Popup infrastructure (glass background)
├── Settings/
│   ├── SettingsWindowController.swift  # NSWindow manager for Settings
│   ├── SettingsView.swift              # Tab-based settings (sidebar nav)
│   ├── PresetEditorView.swift          # Full visual preset editor (sheet)
│   └── Tabs/                           # General, Widgets, Spaces, Time, About
├── Styles/
│   ├── BarStyleProvider.swift          # Protocol + BarStyle enum + @Environment keys
│   ├── GlassStyle.swift                # Liquid glass — blur + highlight + border
│   ├── SolidStyle.swift                # Flat opaque background
│   ├── MinimalStyle.swift              # Transparent, text/icons only
│   └── SystemStyle.swift               # Native macOS .regularMaterial
├── Utils/
│   ├── AppLogger.swift                 # Rotating file logger
│   ├── ExperimentalConfigurationModifier.swift
│   ├── HotkeyManager.swift            # Carbon RegisterEventHotKey (Ctrl+Option+B)
│   ├── FullscreenDetector.swift        # Auto-hide bar on fullscreen apps
│   ├── ImageCache.swift                # Async image caching
│   ├── VersionChecker.swift            # Version tracking for "What's New"
│   └── WindowGapManager.swift          # AX-based window gap enforcement
├── Views/
│   ├── MenuBarView.swift               # Widget registry — routes widget IDs to views
│   ├── BackgroundView.swift            # Bar background
│   ├── OnboardingView.swift            # First-launch welcome (4 pages)
│   └── AppUpdater.swift                # Version-check-only (opens GitHub releases)
└── Widgets/                            # See Widgets table below
```

## Widgets

| Widget ID | Directory | Key API / Notes |
|---|---|---|
| `default.spaces` | `Spaces/` | CGS private API, yabai, AeroSpace. 5 display modes, 4 highlight styles |
| `default.activeapp` | `ActiveApp/` | `NSWorkspace.didActivateApplicationNotification` |
| `default.nowplaying` | `NowPlaying/` | MediaRemote (primary) + AppleScript fallback. Optimistic UI + grace period |
| `default.time` | `Time+Calendar/` | ICU date format, EventKit calendar popup |
| `default.battery` | `Battery/` | IOKit `AppleSmartBattery`, ring popup |
| `default.network` | `Network/` | `getifaddrs` + CoreWLAN, signal/speed/IP popup |
| `default.volume` | `Volume/` | CoreAudio, event-driven, scroll-to-adjust |
| `default.weather` | `Weather/` | MET Norway (default) / OpenMeteo, CoreLocation + IP fallback |
| `default.systemmonitor` | `SystemMonitor/` | `host_statistics` CPU + `vm_statistics64` RAM, 3s poll |
| `default.disk` | `Disk/` | `FileManager` disk stats, 60s poll |
| `default.pomodoro` | `Pomodoro/` | Timer states, local notifications |
| `default.inputlanguage` | `InputLanguage/` | TIS API + DistributedNotificationCenter (zero polling) |
| `default.brightness` | `Brightness/` | CoreDisplay via dlopen, scroll-to-adjust |
| `default.clipboard` | `Clipboard/` | NSPasteboard monitor, 20-entry history |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [azixxxxx/glance](https://github.com/azixxxxx/glance) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-04 -->
