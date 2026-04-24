---
trigger: always_on
description: SAM (Synthetic Autonomic Mind) is a native macOS AI assistant built with Swift and SwiftUI. This guide provides essential information for all AI coding agents (CLIO, Cursor, Aider, GitHub Copilot, etc.).
---

# AGENTS.md - SAM Project Guide

SAM (Synthetic Autonomic Mind) is a native macOS AI assistant built with Swift and SwiftUI. This guide provides essential information for all AI coding agents (CLIO, Cursor, Aider, GitHub Copilot, etc.).

## Quick Facts

- **Language:** Swift 6.0+ (strict concurrency mode enabled)
- **Platform:** macOS 14.0+, Apple Silicon (arm64) preferred, Intel support secondary
- **Build System:** Swift Package Manager (SPM) + Makefile
- **Architecture:** Modular, actor-based concurrency, privacy-first design
- **Testing:** XCTest framework, CI/CD via GitHub Actions
- **Repository:** https://github.com/SyntheticAutonomicMind/SAM
- **License:** GPL-3.0-only

---

## Model Selection

**Use MiniMax for all sub-agents:**
```
agent_operations(operation: "spawn", task: "...", working_dir: "./SAM", model: "minimax/minimax-m2.7")
```

MiniMax-M2.7 via MiniMax is the recommended default for all standard tasks: investigation, QA, implementation, code review, refactoring, documentation.

---

## Setup Commands

### Prerequisites

- macOS 14.0+ (development on macOS 15.0+)
- Xcode 16.0+ with Swift 6.0+
- Command line tools: `xcode-select --install`
- ccache (optional, speeds up builds): `brew install ccache`

### Initial Setup

```bash
# Clone with submodules (required for llama.cpp)
git clone --recursive https://github.com/SyntheticAutonomicMind/SAM.git
cd SAM

# First build automatically builds llama.cpp framework
make build-debug
```

### Build Commands

```bash
# Debug build (faster, includes debug symbols)
make build-debug

# Release build (optimized for production)
make build-release

# Development release (auto-increments -dev.N)
make build-dev

# Clean all build artifacts
make clean

# Build only llama.cpp framework (macOS only)
make llamacpp
```

### Run Commands

```bash
# Run debug build
.build/Build/Products/Debug/SAM.app/Contents/MacOS/SAM

# Run release build
.build/Build/Products/Release/SAM.app/Contents/MacOS/SAM

# Open in Finder
open .build/Build/Products/Debug/SAM.app
```

### Test Commands

```bash
# Run all unit tests
swift test

# Run specific test suite
swift test --filter MyTestSuite

# Run all tests (unit + e2e)
./Tests/run_all_tests.sh

# Run MCP API tests
./Tests/mcp_api_tests.sh

# Run tests like CI/CD pipeline
./scripts/test_like_pipeline.sh
```

---

## Code Style & Conventions

### SPDX License Headers (MANDATORY)

ALL Swift files must begin with:
```swift
// SPDX-License-Identifier: GPL-3.0-only
// SPDX-FileCopyrightText: Copyright (c) 2025 Andrew Wyatt (Fewtarius)
```

### Naming Conventions

- **Classes/Structs/Enums:** `PascalCase` (e.g., `ConversationManager`, `ModelState`)
- **Functions/Variables:** `camelCase` (e.g., `loadModel()`, `conversationHistory`)
- **Constants:** `camelCase`, NOT `SCREAMING_SNAKE_CASE` (e.g., `let maxTokens = 2048`)
- **Protocols:** Descriptive names ending in `Protocol` when needed (e.g., `ToolRegistryProtocol`)
- **Actors:** `PascalCase` with `Actor` suffix when ambiguous (e.g., `ModelLoadingActor`)

### Comments & Documentation

- **Documentation comments:** `///` (shown in Xcode Quick Help)
- **Implementation comments:** `//` (explain WHY, not WHAT)
- **Section markers:** `// MARK: -` to organize code sections
- **Document intent:** Code should be self-documenting; comments explain decisions

### Logging

Use `swift-log` Logger (NEVER `print()` or `NSLog()`):
```swift
private let logger = Logger(label: "com.sam.modulename")

logger.debug("Debug message")
logger.info("Info message")
logger.warning("Warning message")
logger.error("Error message")
```

Logger label format: `com.sam.<module>` (e.g., `com.sam.orchestrator`, `com.sam.conversation`)

---

## Swift 6 Concurrency (CRITICAL - Non-Negotiable)

SAM uses **Swift 6 strict concurrency checking**. All code MUST follow these rules:

### Rules

1. **Sendable Conformance Required**
   - All types crossing actor boundaries must conform to `Sendable`
   - Use `@unchecked Sendable` ONLY when safe, with documentation

2. **MainActor Isolation for UI**
   - All SwiftUI views must be `@MainActor`
   - All ViewModels must be `@MainActor`
   - NSAttributedString operations MUST run on MainActor

3. **Capture Before Crossing Actor Boundaries**
   ```swift
   // ❌ BAD
   await withTaskGroup { group in
       group.addTask { await self.property.doSomething() }
   }
   
   // ✅ GOOD
   let property = self.property
   await withTaskGroup { group in
       group.addTask { await property.doSomething() }
   }
   ```

4. **Expected Build Results**
   - **0 errors** (always)
   - **~211 warnings** (Sendable-related, non-blocking, acceptable)
   - Run `make build-debug` to verify locally
   - Run `./scripts/test_like_pipeline.sh` to simulate CI/CD

### Common Patterns

See `project-docs/SWIFT6_CONCURRENCY_MIGRATION.md` for:
- Wrapper types for non-Sendable dictionaries
- `nonisolated(unsafe)` safe usage
- Capture patterns in loops
- Custom type erasure for Sendable compliance

---

## Project Structure

### Source Modules

```
Sources/
├── SAM/
│   ├── main.swift - Entry point, AppDelegate
│   └── App.swift - SwiftUI App definition

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SyntheticAutonomicMind/SAM](https://github.com/SyntheticAutonomicMind/SAM) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
