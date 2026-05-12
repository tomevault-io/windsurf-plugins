---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Pine is a minimal native macOS code editor built with SwiftUI + AppKit. Targets macOS 26 (Tahoe) with Liquid Glass UI.

**Dependencies** (via Xcode SPM):
- [SwiftTerm](https://github.com/migueldeicaza/SwiftTerm) — terminal emulator
- [Sparkle](https://github.com/sparkle-project/Sparkle) — auto-updates
- [swift-markdown](https://github.com/swiftlang/swift-markdown) — markdown preview rendering

## Build & Run

- **Xcode 26+** required, macOS 26+ deployment target
- Open `Pine.xcodeproj` in Xcode, build and run (Cmd+R)
- CLI build: `xcodebuild -project Pine.xcodeproj -scheme Pine build` (requires `sudo xcode-select -s /Applications/Xcode.app/Contents/Developer`)
- Type-check a single file (no sudo needed): `/Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/swiftc -typecheck -target arm64-apple-macos26.0 -sdk /Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX.sdk <file.swift>`
- **Dependency:** SwiftTerm added via Xcode SPM (File > Add Package Dependencies > `https://github.com/migueldeicaza/SwiftTerm.git`)
- No other third-party dependencies
- **Xcode project format:** Uses `PBXFileSystemSynchronizedRootGroup` (objectVersion 77) — new `.swift` files placed in `Pine/`, `PineTests/`, or `PineUITests/` are automatically picked up by Xcode. No manual `project.pbxproj` edits needed
- **Git hooks:** Run once after cloning: `git config core.hooksPath .githooks && git config merge.ours.driver true`. Enables pre-commit hook that auto-unstages cosmetic-only changes to `Localizable.xcstrings` (Xcode build artifacts) and `ours` merge driver for xcstrings conflicts
- **SwiftLint:** `brew install swiftlint` — runs as a build phase; config in `.swiftlint.yml`. Run `swiftlint` before every commit and fix all warnings/errors. If `swiftlint` crashes with `sourcekitdInProc` error, prefix with `DEVELOPER_DIR=/Applications/Xcode.app/Contents/Developer`
- **Unit Tests:** `DEVELOPER_DIR=/Applications/Xcode.app/Contents/Developer xcodebuild test -project Pine.xcodeproj -scheme Pine -destination 'platform=macOS' -only-testing:PineTests`
- Run a single test class: `DEVELOPER_DIR=/Applications/Xcode.app/Contents/Developer xcodebuild test -project Pine.xcodeproj -scheme Pine -destination 'platform=macOS' -only-testing:PineTests/GoToLineTests`
- Unit test target: `PineTests` (Swift Testing framework) — covers git parsing, grammar models, file tree, syntax highlighting, find & replace, code folding, minimap, status bar, project search, and more (46+ test files)
- **UI Tests:** `DEVELOPER_DIR=/Applications/Xcode.app/Contents/Developer xcodebuild test -project Pine.xcodeproj -scheme Pine -destination 'platform=macOS' -only-testing:PineUITests`
- UI test target: `PineUITests` (XCTest/XCUITest) — end-to-end tests for Welcome window, editor tabs, terminal, multi-window, minimap, git blame, branch switcher, and more (17+ test files)
- Launch arguments for UI testing: `--reset-state` (clears persisted sessions), `-ApplePersistenceIgnoreState YES` (ignores macOS saved window state), `-AppleLanguages (en)`, `-AppleLocale en_US` (force English locale so menu item names are predictable)
- Environment variable for UI testing: `PINE_OPEN_PROJECT=<path>` (opens project without file dialog — uses env var because macOS interprets bare paths in launch arguments as files to open)
- **Known issue:** On macOS 26, `XCUIApplication.launch()` bypasses LaunchServices, so SwiftUI `.defaultLaunchBehavior(.presented)` does not create windows. The app includes an AppKit fallback (`createWelcomeWindowViaAppKit`) that activates after 0.5s if no windows appear.
- **Known issue:** `GutterTextView` (NSTextView inside NSViewRepresentable) does not receive keyboard input from XCUITest's `typeText()`/`typeKey()`. UI tests that need to verify editor content changes should use alternative approaches (e.g., verifying menu item availability, checking tab state).
- **Known issue:** XCUITest's `typeKey()` bypasses the app's `NSEvent.addLocalMonitorForEvents` — synthetic key events go through Accessibility APIs, not the app's event queue. Keyboard shortcuts handled via local event monitors (e.g., Cmd+W for tab closing, Cmd+Shift+B for branch switcher) cannot be reliably UI-tested with `typeKey()`. Use mouse clicks on UI elements instead.
- **Known issue:** SwiftUI's `toolbarTitleMenu` does not work on macOS 26 with Liquid Glass — the title/subtitle area is not clickable. Branch switching uses an AppKit workaround (`BranchSubtitleClickHandler`) that attaches an `NSClickGestureRecognizer` to the subtitle `NSTextField` in the window view hierarchy.
- **Known issue:** UI tests that use `Process()` to run shell commands (e.g., `git init`) need `DEVELOPER_DIR` set in the process environment, otherwise `xcrun` fails with "cannot be used within an App Sandbox".
- To interact with menu items in UI tests, use `app.menuBars.menuBarItems["File"].click()` then `app.menuItems["Item Name"].click()` with English names (locale is forced to `en`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [batonogov/pine](https://github.com/batonogov/pine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
