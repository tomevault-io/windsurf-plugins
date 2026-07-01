---
trigger: always_on
description: automates the rest.
---

# AGENTS.md — app-menu

**Repository**: <https://github.com/barseghyanartur/app-menu>
**Maintainer**: Artur Barseghyan <artur.barseghyan@gmail.com>

---

## 1. Project mission

> A lightweight macOS menu-bar application that surfaces installed
> applications — grouped by their App Store category — via a system status-bar
> menu, with no external dependencies beyond the macOS SDK.

Key constraints that must never be violated:

- **No third-party dependencies.** The app uses only Apple frameworks
  (`AppKit`, `SwiftUI`, `Foundation`).
- **Sandbox-compatible.** All filesystem access must go through the
  security-scoped bookmark API (`DirectoryAccess`). Never bypass the sandbox.
- **Swift 5 / macOS 13.1+.** Do not use APIs that require a higher deployment
  target without gating with `#available`.
- **No network access at runtime.** The app never makes outbound requests.

---

## 2. Repository layout

```text
ApplicationMenu/
    ApplicationMenuApp.swift        # @main entry point, AppDelegate, all views,
                                    # DirectoryAccess, URL.userHome extension
    ContentView.swift               # Unused SwiftUI placeholder (not surfaced in UI)
    SettingsWindowController.swift  # Dead code — legacy NIB-based controller,
                                    # superseded by the SwiftUI SettingsView in
                                    # ApplicationMenuApp.swift. Do not delete yet;
                                    # see §6.
    Info.plist                      # Injects $(MARKETING_VERSION) as AppVersion
    ApplicationMenu.entitlements    # App Sandbox + user-selected read-only files
    Assets.xcassets/                # App icon (tabler icons, MIT licensed)

ApplicationMenuTests/
    ApplicationMenuTests.swift      # Unit tests (XCTest, @testable import)

ApplicationMenuUITests/
    ApplicationMenuUITests.swift        # Basic launch + state assertion
    ApplicationMenuUITestsLaunchTests.swift  # Screenshot on launch

ApplicationMenu.xcodeproj/          # Xcode project; do not hand-edit
CHANGELOG.rst
README.rst
TESTING.md                          # Test running guide
```

The entire application logic lives in **`ApplicationMenuApp.swift`**.  There is
no separate model layer, no persistence layer beyond `UserDefaults`, and no
networking.

---

## 3. Architecture

### 3.1 Class / struct map

| Symbol | Kind | Responsibility |
| --- | --- | --- |
| `URL.userHome` / `URL.userHomePath` | Extension | Resolves `~` via `getpwuid` (sandbox-safe) |
| `DirectoryAccess` | Class (static methods) | Security-scoped bookmark lifecycle |
| `AppDelegate` | `NSObject, NSApplicationDelegate` | Status bar item, menu population, window management |
| `SettingsView` | SwiftUI `View` | Tab container for the three settings panes |
| `AppearanceSettingsView` | SwiftUI `View` | Radio group: Text / Icon / Text & Icon |
| `AppsMenuSettingsView` | SwiftUI `View` | Toggle: case-insensitive sorting |
| `DirectoryAccessView` | SwiftUI `View` | Grant / retract `~/Applications` access |
| `ChromeAppsSettingsView` | SwiftUI `View` | Toggle: show browser web-apps |
| `AboutView` | SwiftUI `View` | Tab container: License, Credits, Version |
| `LicenseView` / `CreditsView` / `VersionView` | SwiftUI `View` | Static info panes |

### 3.2 Menu population flow

```text
applicationDidFinishLaunching
  └─ DirectoryAccess.restoreAccess()   # re-hydrate security-scoped bookmark
  └─ populateMenu()
       ├─ scan /Applications, /System/Applications, ~/Applications
       │    └─ fetchAppDetails(atPath:)
       │         ├─ read Info.plist → CFBundleName / LSApplicationCategoryType
       │         ├─ load .icns icon
       │         └─ fallback: filename → makeHumanReadableFromFilename
       ├─ group apps by makeHumanReadable(LSApplicationCategoryType)
       ├─ sort each group (case-sensitive or -insensitive per UserDefaults)
       ├─ [optional] Chrome/Brave/Edge/Opera/Vivaldi apps submenu
       ├─ "All" flat submenu
       └─ Separator → Refresh / Settings / About / Quit
```

### 3.3 UserDefaults keys

| Key | Type | Default | Used by |
| --- | --- | --- | --- |
| `menuBarOption` | `Int` | `0` | `AppDelegate.configureMenuBarItem()` — 0=Text, 1=Icon, 2=Text+Icon |
| `caseInsensitiveAppsSorting` | `Bool` | `false` | Sort comparator in `populateMenu()` |
| `showChromeApps` | `Bool` | `false` | Chrome-apps submenu block in `populateMenu()` |
| `userSelectedDirectory` | `Data` | absent | Security-scoped bookmark for `~/Applications` |
| `listAppsFromSubDirsRecursively` | `Bool` | `false` | **Not yet active** — toggled UI is commented out |
| `showFavourites` | `Bool` | `false` | FavouritesManager.showFavourites / `@AppStorage` in ApplicationMenuApp |
| `favouriteAppBundleIDs` | `[String]` | `[]` | FavouritesManager.favouriteAppBundleIDs |

### 3.4 Notification

`NSNotification.Name("MenuOptionChanged")` is posted by `SettingsView.saveSettings()`
and observed by `AppDelegate` to trigger `configureMenuBarItem()` without a full
menu rebuild.

---

## 4. Key behaviours and invariants

1. **`DirectoryAccess.restoreAccess()` returns `nil` when no bookmark exists** —
   callers must handle this gracefully (`populateMenu()` is called either way).
2. **`fetchAppDetails` never returns a `nil` icon** — it falls back to
   `NSWorkspace.shared.icon(forFile:)`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [barseghyanartur/app-menu](https://github.com/barseghyanartur/app-menu) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
