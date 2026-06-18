---
trigger: always_on
description: Barly - macOS status bar management utility
---

# Barly - AGENTS.md

## Project Overview

Barly - macOS status bar management utility


## Tech Stack
- **Language**: Swift 6.1
- **UI Framework**: AppKit + SwiftUI (views only)
- **IDE**: Xcode
- **Platforms**: macOS
- **Minimum Deployment**: macOS 15.6 (Sequoia)

## Style & Conventions (MANDATORY)
**Strictly follow** the Swift/SwiftUI style guide: `~/Agents/Style/swift-swiftui-style-guide.md`

## Changelog (MANDATORY)
**All important user facing changes** (fixes, additions, deletions, changes) must be written to CHANGELOG.md.
Changelog format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## Localization (MANDATORY)
**Strictly follow** the localization guide: `~/Agents/Guides/localization-guide.md`
- All user-facing strings must be localized
- Follow formality rules per language
- Consistency is paramount

## Additional Guides
- Modern SwiftUI patterns: `~/Agents/Guides/swift-modern-development-guide.md`
- Observable migration: `~/Agents/Guides/swift-observable-migration-guide.md`
- Swift 6 concurrency: `~/Agents/Guides/swift6-concurrency-guide.md`
- Swift 6 migration (compact): `~/Agents/Guides/swift6-migration-compact-guide.md`
- Swift 6 migration (full): `~/Agents/Guides/swift6-migration-full-guide.md`

## Logging (MANDATORY)
This project uses **DZFoundation** (`~/GIT/Libraries/DZFoundation`) for logging.

**All debug logging must use:**
- `DZLog("message")` — General debug output
- `DZErrorLog(error)` — Conditional error logging (only prints if error is non-nil)

```swift
import DZFoundation

DZLog("Starting fetch")       // 🔶 fetchData() 42: Starting fetch
DZErrorLog(error)             // ❌ MyFile.swift:45 fetchData() ERROR: Network unavailable
```

**Do NOT use:**
- `print()` for debug output
- `os.Logger` instances
- `NSLog`

Both functions are no-ops in release builds.

## API Documentation
Local Apple API documentation is available at:
`~/Agents/API Documentation/Apple/`

The `search` binary is located **inside** the documentation folder:
```bash
~/Agents/API\ Documentation/Apple/search --help  # Run once per session
~/Agents/API\ Documentation/Apple/search "view controller" --language swift
~/Agents/API\ Documentation/Apple/search "NSWindow" --type Class
```

## Xcode Project Files (CATASTROPHIC — DO NOT TOUCH)
- **NEVER edit Xcode project files** (`.xcodeproj`, `.xcworkspace`, `project.pbxproj`, `.xcsettings`, etc.)
- Editing these files will corrupt the project — this is **catastrophic and unrecoverable**
- Only the user edits project settings, build phases, schemes, and file references manually in Xcode
- If a file needs to be added to the project, **stop and tell the user** — do not attempt it yourself
- Use `xcodebuild` for building/testing only — never for project manipulation
- **Exception**: Only proceed if the user gives explicit permission for a specific edit
  
## File System Synchronized Groups (Xcode 16+)
This project uses **File System Synchronized Groups** (internally `PBXFileSystemSynchronizedRootGroup`), introduced in Xcode 16. This means:
- The `Classes/` and `Resources/` directories are **directly synchronized** with the file system
- **You CAN freely create, move, rename, and delete files** in these directories
- Xcode automatically picks up all changes — no project file updates needed
- This is different from legacy Xcode groups, which required manual project file edits

**Bottom line:** Modify source files in `Classes/` and `Resources/` freely. Just never touch the `.xcodeproj` files themselves.

## Code Formatting (MANDATORY)
**Always run SwiftFormat after a successful build:**
```bash
swiftformat .
```

SwiftFormat configuration is defined in `.swiftformat` at the project root. This enforces:
- 4-space indentation
- Explicit `self.` usage
- K&R brace style
- Trailing commas in collections
- Consistent wrapping rules

**Do not commit unformatted code.**

## Naming Convention

- **Menu Bar**: The left side of the top bar showing app menus (App Name, File, Edit, View, etc.)
- **Status Bar**: The right side of the top bar where icons live (system icons, third-party app icons)

## Build Commands

```bash
# Build from command line
xcodebuild -project src/Barly.xcodeproj -scheme Barly -configuration Debug build

# Run tests
xcodebuild -project src/Barly.xcodeproj -scheme Barly -configuration Debug test

# Clean build
xcodebuild -project src/Barly.xcodeproj -scheme Barly clean
```

Open `src/Barly.xcodeproj` in Xcode to build and run directly.

## Architecture

Barly is a macOS status bar management app that hides/shows status bar items. It uses a native AppKit lifecycle (`main.swift` + `NSApplication`) with SwiftUI for views only (hosted via `NSHostingController`).

### Core Flow

1. **main.swift** (`src/Barly/Classes/main.swift`) - Native AppKit entry point, creates `NSApplication` and sets `AppDelegate`
2. **AppDelegate** (`src/Barly/Classes/AppDelegate.swift`) - Initializes `StatusBarController` and `HotkeyManager` on launch
2. **AppDelegate** (`src/Barly/Classes/AppDelegate.swift`) - Initializes `StatusBarController` and `HotkeyManager` on launch
3. **StatusBarController** (`src/Barly/Classes/Controllers/StatusBarController.swift`) - Core logic for the status bar items


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [domzilla/Barly](https://github.com/domzilla/Barly) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
