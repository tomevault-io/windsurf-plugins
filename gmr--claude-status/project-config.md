---
trigger: always_on
description: Claude Status is a native macOS menu bar app that monitors active Claude Code sessions on the local machine. It shows session state, project info, and provides one-click focus to the session's host app (terminals and IDEs). Distributed outside the Mac App Store via Developer ID signing, notarization, and Sparkle auto-updates.
---

# CLAUDE.md

## Project Overview

Claude Status is a native macOS menu bar app that monitors active Claude Code sessions on the local machine. It shows session state, project info, and provides one-click focus to the session's host app (terminals and IDEs). Distributed outside the Mac App Store via Developer ID signing, notarization, and Sparkle auto-updates.

## Build & Test

Xcode project with SPM dependencies (no standalone Package.swift). Use `just` for all build and test commands:

```bash
just build          # Build debug configuration (includes plugin binaries)
just test           # Run all unit tests
just test-class SessionStateTests  # Run a single test class
just clean          # Clean build artifacts
just swap           # Build, copy to /Applications, and relaunch
just sync-plugin    # Sync plugin to installed plugin cache
just show-version   # Show calculated version from git tags
```

The justfile handles `MACOSX_DEPLOYMENT_TARGET=15.0` override (needed for Xcode versions that don't know about macOS 26.2) and disables code signing for local dev builds.

## Platform & Language

- **macOS 26.2+** deployment target (override to 15.0 for CI/older Xcode)
- **Swift 5.0** with `SWIFT_APPROACHABLE_CONCURRENCY = YES`, `SWIFT_DEFAULT_ACTOR_ISOLATION = MainActor`
- **SwiftUI + AppKit** hybrid: AppKit for `NSStatusItem`/`NSPopover`/`NSWindow`, SwiftUI for all views
- **Menu bar-only**: `LSUIElement = YES` (no Dock icon)
- **No App Sandbox**: required for `proc_pidinfo`, `sysctl KERN_PROCARGS2`, and AppleScript automation
- Bundle ID: `com.poisonpenllc.Claude-Status`
- Widget Bundle ID: `com.poisonpenllc.Claude-Status.widget`
- App Group: `group.com.poisonpenllc.Claude-Status` (shared data between app and widget)
- URL Scheme: `claude-status://`

## Dependencies (SPM)

| Package | Version | Purpose |
|---------|---------|---------|
| Sparkle | 2.9.0 | Auto-updates via Appcast (EdDSA-signed) |
| CocoaLumberjack | 3.9.0 | Structured logging |
| swift-log | 1.10.1 | Swift logging API |

## Architecture

### Targets

| Target | Bundle ID | Purpose |
|--------|-----------|---------|
| `Claude Status` | `com.poisonpenllc.Claude-Status` | Main app: `NSStatusItem` + `NSPopover` with SwiftUI views |
| `Claude StatusTests` | `com.poisonpenllc.Claude-StatusTests` | Unit tests (Swift Testing framework) |
| `Claude StatusWidgetExtension` | `com.poisonpenllc.Claude-Status.widget` | WidgetKit desktop widgets (status, productivity, score) |

### Source Layout

```
Claude Status/                         # Main app target
  AppMain.swift                        # Entry point, menu bar-only setup
  AppDelegate.swift                    # NSStatusItem, NSPopover, settings window, Sparkle updater
  Claude_StatusWidgetConfiguration.swift  # WidgetKit configuration
  Info.plist                           # Sparkle feed URL, URL scheme
  Claude Status.entitlements           # App Groups (no sandbox)
  SessionDiscovery/                    # Core session monitoring
    SessionDiscovery.swift             # Scans ~/.claude/projects/*/*.cstatus, validates PIDs, classifies source
    SessionMonitor.swift               # @Observable class: Darwin notifications + file watching + 5s polling
    StateResolver.swift                # DispatchSource file watcher; JSONL timestamp fallback
    ITermFocuser.swift                 # Focuses host app (AppleScript for iTerm2, process activation for others)
    ProductivityTracker.swift          # Time-in-state tracking, concurrency, score (persists to App Group)
    PluginDetector.swift               # Checks installed_plugins.json and settings.json for hook status
    PluginInstaller.swift              # Installs/uninstalls bundled plugin via `claude plugin` CLI
  Views/                               # SwiftUI views
    SessionListView.swift              # Popover: header, session list, empty state, Settings/Quit
    SessionRowView.swift               # Session row: status icon, project, source, activity, time
    SettingsView.swift                 # Icon style picker, launch at login, plugin management
    ProductivityBarView.swift          # Visual productivity tracking bar

Shared/                                # Models shared between app and widget
  ClaudeSession.swift                  # ClaudeSession model, SessionState enum, SessionSource enum
  ProductivityStats.swift              # ProductivityStats and ProductivityData models

Claude StatusWidget/                   # Widget extension target
  Claude_StatusWidget.swift            # Widget bundle (3 widgets)
  Claude_StatusTimelineProvider.swift   # WidgetKit timeline provider
  Claude_StatusWidgetEntryView.swift   # Session status widget view
  ProductivityWidget.swift             # Productivity widget definition
  ProductivityWidgetView.swift         # Productivity widget view
  ScoreWidgetView.swift                # Score visualization widget view
  Info.plist                           # Extension metadata

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gmr/claude-status](https://github.com/gmr/claude-status) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
