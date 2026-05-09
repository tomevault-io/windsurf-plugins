---
trigger: always_on
description: CodeLooper project guidance for AI assistants
---

# Agent Instructions

This file provides guidance to AI assistants when working with code in this repository.

## General useful for debug loops:
- The Claude Code tool is helpful to analyze large logs or do complex file edits.
- Pipe scripts that output lots of logs into a file for easier reading.
- Use AppleScript to execute apps such as Claude Code to test the mcp.
(This requires that the mcp is correctly set up)
- Whenever you want to ask something to the user, ask Claude Code first instead.
- Use AppleScript to find the bundle identifier for an app name
- Use `jq` to analyze large json files.
- We do not need backward compatibility, refactor properly.
- We use Swift 6 for the CLI.

- When I say "regenerate" then use `./scripts/generate-xcproj.sh` to regen the xcodeproj and use `xcodebuild` to test that it builds.
- "fix build" means using `xcodebuild` with `CodeLooper.xcodeproj`. When you added/removed/changed files, regenerate first.
- You don't need to regenerate the Xcode project as long as you just make modifications in one file. You only need to do this if you add or remove files. 

## Project Overview

This is the **CodeLooper** project, a Swift macOS menu bar application that provides automation and workflow enhancement capabilities. The application focuses on:

- Menu bar presence with status icon management
- User settings and preferences management  
- macOS accessibility automation via AXorcist integration
- Comprehensive logging and diagnostics system
- SwiftUI-based settings interface with AppKit foundation
- AXorcist is based on C-API that is synchronous and main-thread only. Therefore basically nothing in there should use async.
- Refactor and improve code while you fix up things, don't go for the easy route, do it properly.

## Architecture

- **Swift Package Manager (SPM)**: Primary dependency management and build system
- **Tuist**: Project generation and workspace management. Tuist uses a `Project.swift` manifest file to define the project structure, dependencies, and build settings. The main project configuration can be found in `Project.swift` at the root of the repository.
- **SwiftUI + AppKit**: Hybrid UI approach with SwiftUI for modern components and AppKit for system integration
- **AXorcist Integration**: Accessibility automation framework (git submodule)
- **Swift 6.0**: Strict concurrency checking enabled

### Key Directories

- **`Sources/Application/`**: Core app lifecycle, delegates, and coordination
  - `AppDelegate.swift`: Main application delegate with menu management
  - `AppMain.swift`: Application entry point and setup
  - `WindowPositionManager.swift`: Window management utilities
- **`Sources/Components/`**: UI components and SwiftUI views
  - `SwiftUI/`: Modern SwiftUI-based interface components
  - `Alert/`: Alert presentation system
- **`Sources/Diagnostics/`**: Comprehensive logging and error handling
  - `Logger.swift`: Main logging interface
  - `DiagnosticsLogger.swift`: Advanced diagnostic capabilities
  - `LogManager.swift`: Log configuration and management
- **`Sources/Settings/`**: User preferences and configuration
  - `Models/DefaultsManager.swift`: UserDefaults management
  - `Views/`: Settings UI components
- **`Sources/StatusBar/`**: Menu bar interface and icon management
  - `MenuManager.swift`: Main menu coordination
  - `StatusIcon/`: Icon animation and state management
- **`Sources/Utilities/`**: Shared utilities, extensions, and helpers
  - `Extensions/`: Swift and Foundation extensions
  - `Resources/`: Resource loading and management
- **`AXorcist/`**: Accessibility automation framework (git submodule)

## Development Workflow

### Essential Commands

```bash
# Project setup and building
swift build                           # Build with SPM (less common for day-to-day Xcode work)
./scripts/generate-xcproj.sh         # CRITICAL: Generate/regenerate Xcode project via Tuist. ALWAYS use this script.
                                      # This script includes vital patches for Swift 6 Sendable compliance with Tuist-generated code.
./scripts/open-xcode.sh             # Open in Xcode (use after generating project)
./scripts/run-app.sh                # Run the application (preferably from Xcode after project generation)

# Code quality and formatting
./run-swiftlint.sh                  # Check code quality (preserves self. references)
./run-swiftformat.sh                # Format code automatically
swift test                          # Run unit tests

# AXorcist accessibility testing
cd AXorcist && swift build          # Build accessibility tools
./AXorcist/.build/debug/axorc --help # Test accessibility CLI
```

### Tuist Project Generation

Due to Swift 6's strict concurrency checking and how Tuist generates code (specifically for Plist constants like `cfBundleURLTypes`),
**it is essential to always use the `./scripts/generate-xcproj.sh` script to generate or regenerate the Xcode project.**

This script performs the following crucial steps:
1. Runs `tuist generate` to create the base Xcode project.
2. **Applies necessary patches** to Tuist-generated Swift files (e.g., `Derived/Sources/TuistPlists+CodeLooper.swift`).
   These patches modify the generated code to ensure Sendable compliance, for instance by:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [steipete/CodeLooper](https://github.com/steipete/CodeLooper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
