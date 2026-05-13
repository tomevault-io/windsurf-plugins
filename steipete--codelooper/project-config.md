---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

CodeLooper is a macOS menubar application that monitors Cursor IDE instances and automatically handles stuck states, connection errors, and other interruptions to maintain productive AI-assisted coding sessions. The project uses Swift 6 with strict concurrency checking, SwiftUI/AppKit hybrid architecture, and integrates with the AXorcist accessibility framework for UI automation.

- This project uses Swift 6 with strict concurrency.
- It supports macOS 15+.
- Refactor properly, do not care about backwards compatibility.
- After changing swift files, compile with xcodebuild to verify that everything works. (decide if regular or test build depending on which files you changed.)

- To view log entries from the process as it runs, use: 
  `/usr/bin/log show --process CodeLooper --style compact --last 5m | tail -n 50`
  (5m means 5 minutes, tail -n 50 means last 50 entries)

## Essential Commands

### Building and Running

```bash
# Generate Xcode project (CRITICAL: Always use this script, not 'tuist generate' directly)
./scripts/generate-xcproj.sh

# Open in Xcode
./scripts/open-xcode.sh

# Build with xcodebuild (after regenerating if files were added/removed)
xcodebuild -workspace CodeLooper.xcworkspace -scheme CodeLooper -configuration Debug build

# Run the app from command line
./scripts/run-app.sh

# Build AXorcist tools
cd AXorcist && swift build
```

### Code Quality and Linting

```bash
# Run SwiftLint (preserves self. references for Swift 6)
./run-swiftlint.sh

# Run SwiftFormat
./run-swiftformat.sh

# Run both linting and formatting
./lint.sh
```

### Testing

```bash
# Run Swift tests
swift test

# Run AXorcist tests
cd AXorcist && ./run_tests.sh

# Test AXorcist CLI
./AXorcist/.build/debug/axorc --debug '{"command_id":"test","command":"ping"}'
```

## High-Level Architecture

### Project Structure

The project uses a hybrid build system:
- **Tuist**: Project generation and workspace management (Project.swift)
- **Swift Package Manager**: Dependency management (Package.swift)
- **Swift 6**: Strict concurrency checking enabled throughout

### Key Architectural Components

The project follows a **feature-based architecture** with vertical slicing:

1. **App Layer** (`App/`)
   - `CodeLooperApp.swift`: SwiftUI app entry point
   - `AppDelegate.swift`: Legacy AppKit delegate for system integration
   - Application-level infrastructure and resources
   - Thread-safe with `@MainActor` isolation

2. **Features** (`Features/`) - Each feature is self-contained:
   - **Monitoring** (`Features/Monitoring/`): Core monitoring functionality
     - Domain: `CursorMonitorService`, instance models
     - UI: Monitoring views and status displays
     - Infrastructure: Window observers, lifecycle management
   
   - **Intervention** (`Features/Intervention/`): Automated recovery
     - Domain: `InterventionEngine`, heuristics
     - Strategies: Connection errors, stuck states, file conflicts
     - Infrastructure: Intervention execution
   
   - **AIAnalysis** (`Features/AIAnalysis/`): AI-powered diagnostics
     - Domain: Analysis services, screenshot analyzer
     - Infrastructure: OpenAI/Ollama providers
     - UI: Analysis configuration and results
   
   - **Settings** (`Features/Settings/`): User preferences
     - Domain: Settings service, preference models
     - UI: Settings tabs and coordinator
     - Infrastructure: User defaults storage
   
   - Other features: StatusBar, GitTracking, MCPIntegration, Onboarding

3. **Core** (`Core/`) - Shared functionality:
   - **Accessibility**: AX framework integration, permissions
   - **Diagnostics**: Logging system with categories
   - **JSHook**: JavaScript injection for Cursor UI
   - **Utilities**: Extensions, helpers, constants

4. **External Packages** (`AXorcist/`, `AXpector/`, `DesignSystem/`)
   - Local SPM packages for modularity
   - AXorcist: Accessibility automation framework
   - AXpector: Accessibility inspector
   - DesignSystem: UI components and theming

### Concurrency Model

- **Swift 6 Strict Concurrency**: Complete checking enabled
- **@MainActor**: All UI code and most managers
- **Explicit self.**: Required in closures for capture semantics
- **Sendable Compliance**: Custom types marked appropriately
- **Actor Isolation**: Long-running operations use custom actors

## Critical Development Notes

### Tuist and Swift 6 Sendable Compliance

**ALWAYS use `./scripts/generate-xcproj.sh` instead of `tuist generate`**

The script performs critical patches for Swift 6 compatibility:
1. Fixes Tuist-generated `TuistPlists+CodeLooper.swift` for Sendable compliance
2. Converts `[String: Any]` to type-safe alternatives
3. Updates `ResourceLoader.swift` to handle typed dictionaries

### AXorcist and Electron Apps

When working with Cursor (Electron app) accessibility:
- Electron limits accessibility tree depth (~30-40 nodes)
- Use focus-based queries for deep elements
- The `debugloop.mdc` file contains extensive learnings about Cursor UI traversal
- Key flags: `--scan-all`, `--no-stop-first`, `--timeout`

### SwiftLint Configuration


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [steipete/CodeLooper](https://github.com/steipete/CodeLooper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
