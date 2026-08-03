---
trigger: always_on
description: Guidance for AI Agents working in this repository.
---

# AGENTS.md

Guidance for AI Agents working in this repository.

## Scope

This is a mixed Xcode + SwiftPM project for a Cantonese input method adopting the Jyutping romanization scheme:

- `Jyutping/`: the SwiftUI reference app for iOS, macOS, and visionOS-compatible app builds
- `Keyboard/`: the iOS/iPadOS keyboard extension
- `InputMethod/`: the macOS InputMethodKit target
- `Modules/`: local Swift packages shared by the app and input targets
- `packaging/`: macOS package resources and installer scripts
- `ci_scripts/` and `.github/workflows/`: CI support

## Formatting and editing rules

Read `.editorconfig` before editing.

Current editor settings are UTF-8, LF line endings, and for Swift files: spaces, indent size 8, tab width 8, trim trailing whitespace, and insert a final newline.

Make narrow, surgical edits and follow the surrounding style instead of reformatting files wholesale.

## What is in the project

`xcodebuild -project Jyutping.xcodeproj -list` currently reports these Xcode targets:

- `Jyutping`
- `JyutpingTests`
- `JyutpingUITests`
- `Keyboard`
- `InputMethod`
- `InputMethodTests`
- `InputMethodUITests`

It currently exposes these schemes:

- `AboutKit`
- `AppDataSource`
- `CommonExtensions`
- `CoreIME`
- `InputMethod`
- `Jyutping`
- `Keyboard`
- `Linguistics`

Under `Modules/`, the local Swift packages are:

1. `CommonExtensions`: shared Foundation-style helpers and extensions
2. `CoreIME`: the core input engine and bundled SQLite lexicon
3. `Linguistics`: Jyutping/IPA and related language helpers
4. `AppDataSource`: searchable reference datasets used by the app
5. `AboutKit`: about/info UI support
6. `Preparing`: a SwiftPM-only build-time executable that generates data used by other modules

## Build requirements and environments
- Current local environment observed while updating this file: macOS 27.0, Xcode 27.0, Apple Swift 6.4.
- Package manifests use `swift-tools-version: 6.3` and `swiftLanguageModes: [.v6]`.
- Xcode project settings use Swift 6 for the app, keyboard, input method, and project-level settings.
- The `Preparing` package declares macOS 26+ because it is a local database-generation tool.

Targeted platforms:
- iOS/iPadOS 16.0+
- macOS 13+ (Ventura or above)

## First build step: generate databases

Before building the Xcode project, generate the packaged databases:

```bash
cd Modules/Preparing
swift run -c release
```

This is not optional for a clean checkout. The executable entry point is `Modules/Preparing/Sources/Preparing/Preparing.swift`, which runs `AppDataPreparer.prepare()` and `DatabasePreparer.prepare()` concurrently. It generates these packaged SQLite databases:

- `Modules/CoreIME/Sources/CoreIME/Resources/ime.sqlite3`
- `Modules/AppDataSource/Sources/AppDataSource/Resources/app.sqlite3`

## Runtime architecture

### App target (`Jyutping/`)

- Entry point: `Jyutping/JyutpingApp.swift`
- The app has separate `iOS/` and `macOS/` trees plus shared views/models.
- Code under `Jyutping/iOS/Search`, `Jyutping/macOS/Search`, `Jyutping/iOS/Cantonese`, `Jyutping/macOS/Metro`, `Jyutping/CantoneseMaterials`, and `Jyutping/SharedViews` is the main search/reference UI surface.
- Imports across the app show that it primarily consumes `AppDataSource`, `Linguistics`, `CommonExtensions`, and `AboutKit`.

Useful places:

- `Jyutping/SharedModels/AppMaster.swift`
- `Jyutping/iOS/Home/`
- `Jyutping/iOS/Search/`
- `Jyutping/iOS/Romanization/`
- `Jyutping/iOS/Cantonese/`
- `Jyutping/macOS/Search/`
- `Jyutping/macOS/Metro/`

### iOS keyboard extension (`Keyboard/`)

- Entry point: `Keyboard/KeyboardViewController.swift`
- The controller prepares the keyboard UI, calls `InputMemory.prepare()`, then `Engine.prepare()`, and hosts `MotherBoard`.
- Shared keyboard UI is under `Keyboard/SharedViews/`.
- Keyboard state, layouts, and behavior enums live under `Keyboard/SharedModels/`.
- Device/layout-specific keyboards are split across `iPhone/`, `iPad/`, `NineKey/`, and `SpecialLayouts/`.
- The iPad implementation has separate large, medium, and small key/keyboards folders. The keyboard target also includes emoji, editing-panel, speech, image, and shape support.

Useful places:

- `Keyboard/KeyboardViewController.swift`
- `Keyboard/SharedModels/KeyboardInterface.swift`
- `Keyboard/SharedModels/InputMemory.swift`
- `Keyboard/SharedViews/MotherBoard.swift`
- `Keyboard/SharedViews/CandidateBoard.swift`

### macOS input method (`InputMethod/`)

- SwiftUI app entry point: `InputMethod/InputMethodApp.swift`
- App delegate and IMK server setup: `InputMethod/AppDelegate.swift`
- Main controller: `InputMethod/JyutpingInputController.swift`
- The controller activates the IME server, prepares `InputMemory` and `Engine`, manages the candidate window, and reacts to selection/highlight notifications.
- Candidate UI lives in `InputMethod/CandidateViews/`, `InputMethod/CandidateWindow.swift`, and `InputMethod/MotherBoard.swift`.
- Preferences UI lives in `InputMethod/Preferences/`.
- The target also links Sparkle for update support; see `Sparkle.framework` and the app delegate.

Useful places:

- `InputMethod/JyutpingInputController.swift`
- `InputMethod/InputContext.swift`
- `InputMethod/Models/InputMemory.swift`
- `InputMethod/MotherBoard.swift`
- `InputMethod/CandidateViews/`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yuetyam/jyutping](https://github.com/yuetyam/jyutping) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
