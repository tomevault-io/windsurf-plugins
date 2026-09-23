---
trigger: always_on
description: DeepSeek Monitor is a macOS menu bar app for monitoring DeepSeek account balance,
---

# DeepSeek Monitor - Project guide

DeepSeek Monitor is a macOS menu bar app for monitoring DeepSeek account balance,
token usage, model costs, and recent usage trends. It also provides a native
WidgetKit widget, official usage-export import, silent browser-assisted syncing,
and manually initiated signed software updates.

This is the repository's only agent instruction file. Do not recreate
`CLAUDE.md` or maintain a second copy of this guide.

## Platform and dependencies

- Swift 5.9+, SwiftUI, AppKit, WidgetKit, WebKit, Security, and ServiceManagement.
- macOS 14 or later; release builds target both Apple Silicon and Intel.
- Sparkle 2.9.5 is the only third-party runtime dependency.
- No storyboards or XIB files; application UI is programmatic.
- `LSUIElement = true`: the app is menu-bar-only and hidden from the Dock.
- App bundle ID: `com.deepseek.monitor`.
- Widget bundle ID: `com.deepseek.monitor.widget`.
- App Group: `N5YV5FV235.group.com.deepseek.monitor`.

Treat these files as the version source of truth:

- `build.sh`: marketing version used by release tooling.
- `Resources/Info.plist`: app marketing version and build number.
- `Sources/WidgetSupport/Info.plist`: widget marketing version and build number.

The app and widget versions must always match. Do not hard-code the current
release version elsewhere in this guide.

## Architecture

```text
AppDelegate -> MenuBarManager -> FloatingPanel / SettingsWindow / ModelDetailWindow
            -> DashboardViewModel -> DeepSeekService -> APIKeyStore -> Keychain
                                  -> UsageCSVImporter / UsageAutoImportService
                                  -> LocalCache -> App Group -> WidgetSupport
            -> UsageExportAutomationService -> WKWebView -> official usage ZIP
            -> SoftwareUpdateController -> Sparkle -> signed appcast.xml
```

- `DashboardViewModel` owns refresh scheduling, visible dashboard state, usage
  aggregation, import orchestration, cache writes, and widget snapshots.
- Balance comes from the DeepSeek API. The `/v1/usage` endpoint may return 404;
  usage then comes from official ZIP/CSV exports.
- `UsageExportAutomationService` reuses the user's DeepSeek web session and keeps
  scheduled exports hidden. The login window is shown only for an explicit login
  action or when the user needs to restore the session.
- `WidgetSupport` reads the App Group snapshot and never reads the API key.

## Key files

| File | Responsibility |
|---|---|
| `Sources/DeepSeekMonitor/App.swift` | App entry point, sleep/wake handling, and deep-link dispatch. |
| `Sources/DeepSeekMonitor/MenuBarManager.swift` | Status item, main panel, settings/detail routing, hover behavior, and status menu. |
| `Sources/DeepSeekMonitor/ViewModels/DashboardViewModel.swift` | Refresh, aggregation, cache, import, and widget synchronization. |
| `Sources/DeepSeekMonitor/Services/DeepSeekService.swift` | Balance/usage API requests and in-process API-key access. |
| `Sources/DeepSeekMonitor/Services/APIKeyStore.swift` | Keychain storage and verified legacy migration. |
| `Sources/DeepSeekMonitor/Services/UsageExportAutomationService.swift` | Official-site WKWebView login, silent export, and download handling. |
| `Sources/DeepSeekMonitor/Services/UsageAutoImportService.swift` | ZIP/CSV preparation, archive validation, quarantine, and automatic import state. |
| `Sources/DeepSeekMonitor/Services/UsageCSVImporter.swift` | Official amount/cost schema parsing and aggregation. |
| `Sources/DeepSeekMonitor/Services/SoftwareUpdateController.swift` | Manual Sparkle update checks and user-visible update state. |
| `Sources/DeepSeekMonitor/Services/LocalCache.swift` | Dashboard cache and WidgetKit App Group snapshot. |
| `Sources/DeepSeekMonitor/Views/ContentView.swift` | Main menu bar dashboard. |
| `Sources/DeepSeekMonitor/Views/SettingsView.swift` | API key, widget, login item, update, refresh, and import/export settings. |
| `Sources/DeepSeekMonitor/Views/ModelDetailWindowController.swift` | V4.1 Flash/V4 Flash model detail side panel. |
| `Sources/WidgetSupport/TimelineProvider.swift` | Widget timeline provider reading shared data. |
| `Sources/WidgetSupport/WidgetViews.swift` | Medium WidgetKit UI and deep links. |
| `Resources/Assets.xcassets/DeepSeekMenuBarTemplate.imageset/` | Native 1x/2x template menu bar icon. |
| `Resources/Info.plist` | App identity, version, URL scheme, and Sparkle trust configuration. |
| `appcast.xml` | Sparkle-signed published update feed. |
| `.github/workflows/ci.yml` | Tests, unsigned release build, trust checks, and menu icon validation. |
| `build.sh` | Version bump, Xcode build, signing, cleanup, DMG, and appcast packaging. |

## Build and verify

```bash
swift test
./build.sh run
./build.sh release
./build.sh appcast
./build.sh signed-release
```

- `swift test` runs the package tests without creating release artifacts.
- `./build.sh run` increments the build number, creates a stable development-signed
  Xcode Debug app, verifies its Team ID, and opens it.
- `./build.sh release` increments the build number, builds the universal app and
  widget, signs nested Sparkle components, verifies the bundle, and creates the DMG.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JayHome137/DeepSeekMonitor](https://github.com/JayHome137/DeepSeekMonitor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
