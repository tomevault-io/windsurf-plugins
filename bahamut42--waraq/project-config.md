---
trigger: always_on
description: Developer-facing notes for a Claude Code session working inside this repo. End users should read `README.md` instead.
---

# CLAUDE.md — Waraq

Developer-facing notes for a Claude Code session working inside this repo. End users should read `README.md` instead.

Waraq is a shipped macOS 14+ animated wallpaper app (Swift 5.10, SwiftUI + AppKit). v1.0.0 is released. Treat the codebase as production: any change should keep the existing tests green and the signed-and-notarized release pipeline reproducible.

## Repo layout

```
App/         SwiftUI + AppKit shell: AppDelegate, menu bar, Settings panes, onboarding wizard
Core/        Cross-engine logic: DisplayManager, WallpaperLibrary, PerformanceGovernor, ResourceMonitor, Gallery clients (Pixabay/Pexels/NASA), WaraqPrimaryStore
Engines/     Per-wallpaper render engines: VideoEngine, GifEngine, GradientWallpaper, Procedural/*
Library/     LibraryView (a UI surface, not the storage layer)
Resources/   Assets.xcassets (app icon, menu bar icon)
Scripts/     One-off Swift scripts (GenerateAppIcon, GenerateMenuBarIcon). Not built into the app.
Tests/       XCTest unit tests (WaraqPrimaryStoreTests, GalleryTests, ProceduralThumbnailTests, WaraqTests)
docs/        design/ specs per pane + RELEASE_NOTES_v1.0.0*.md + AUDIT_REPORT.md + install/ (GitHub Pages site)
.github/workflows/build.yml   CI: macos-latest, xcodebuild build + test on push/PR to main
project.yml  XcodeGen spec — source of truth for the Xcode project
Screensaver/ Reserved (empty). The screensaver target referenced in PHASE0_HANDOFF.md was deferred past v1.
```

## How the project is generated

`Waraq.xcodeproj/` is in `.gitignore`. Do not commit it. Regenerate locally:

```bash
brew install xcodegen swiftlint swiftformat
xcodegen generate
```

If you edit `project.yml` (targets, sources, build settings, Info.plist values), rerun `xcodegen generate` before building.

## Build, run, test

Open in Xcode after generating:

```bash
open Waraq.xcodeproj
```

Cmd+B builds, Cmd+R runs, Cmd+U tests.

Command-line build (matches CI in `.github/workflows/build.yml`):

```bash
xcodebuild \
  -project Waraq.xcodeproj \
  -scheme Waraq \
  -destination 'platform=macOS' \
  -configuration Debug \
  CODE_SIGNING_ALLOWED=NO \
  build
```

Command-line tests:

```bash
xcodebuild \
  -project Waraq.xcodeproj \
  -scheme Waraq \
  -destination 'platform=macOS' \
  -configuration Debug \
  CODE_SIGNING_ALLOWED=NO \
  test
```

Per the v1.0.0 audit, the test suite is 54/54 passing. Don't merge code that drops that count.

## Lint and format

```bash
swiftlint
swiftformat --lint .
```

Both are wired by `.swiftlint.yml` and `.swiftformat`. CI doesn't currently fail on lint violations, but the v1.0.0 audit gate did, so keep both clean.

Notable rules from `.swiftlint.yml`:

- `line_length` warning 140, error 200
- `file_length` warning 500, error 800
- `function_body_length` warning 60, error 120
- `type_body_length` warning 350, error 500
- `cyclomatic_complexity` warning 12, error 20

## Architecture in brief

```
AppDelegate
  └── DisplayManager (one per app launch, @MainActor singleton-like)
        ├── per-display WallpaperWindow  (borderless NSWindow at desktopIconWindow-1 level, ignoresMouseEvents)
        ├── per-display engine instance  (VideoEngine | GifEngine | GradientWallpaper | procedural NSView)
        ├── WallpaperLibrary             (~/Library/Application Support/Waraq/{Wallpapers,Thumbnails,library.json})
        ├── PerformanceGovernor          (publishes per-display PlaybackState; reacts to battery, fullscreen, thermal)
        └── ResourceMonitor              (CPU/GPU/RAM sampling for Diagnostics pane)
  └── MenuBarController                  (NSStatusItem + NSPopover hosting MenuBarPopoverView)
```

Key invariants:

- `WallpaperWindow` sits one level below the desktop-icon window so icons stay clickable on top. Do not move it above.
- `DisplayManager.applyGovernorState(_:)` early-returns if `isPaused == true` so the menu-bar pause toggle wins over governor decisions.
- Display profiles are keyed by hardware ID via `DisplayProfile` / `WaraqPrimaryStore`, not by `CGDirectDisplayID` (which churns across reboots).

## Gallery and external network policy

Privacy is a contract with users. The README states zero outbound traffic unless the user explicitly searches the Gallery. Code changes must respect this:

- The only network egress points are `Core/Gallery/PixabayClient.swift`, `PexelsClient.swift`, `NASAClient.swift`, and `GalleryDownloader.swift`.
- "Browse Web" cards must continue to open URLs in the user's default browser via `NSWorkspace.open`. Do not fetch, scrape, mirror, or proxy content from MotionBGs, MoeWalls, MyLiveWallpapers, or Wallsflow.
- API keys (Pixabay, Pexels) live in `APIKeyStore.swift` (UserDefaults). NASA needs no key.
- No telemetry. No analytics. No phone-home. Do not add any.

## License header

Every Swift source file in `App/`, `Core/`, `Engines/`, `Library/`, `Tests/`, `Scripts/` should start with the GPL v3 boilerplate header. The audit script counts files vs. files-with-header. When adding a new `.swift` file, copy the header from any existing file verbatim.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bahamut42/waraq](https://github.com/bahamut42/waraq) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-29 -->
