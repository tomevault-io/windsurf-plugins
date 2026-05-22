---
trigger: always_on
description: This file provides guidance to AI coding agents when working with code in this project.
---

# AI Coding Agent Rules, Operating Procedures, Guidelines and Core Project Memory

This file provides guidance to AI coding agents when working with code in this project.


--- 


## Project Overview

A Swift command-line tool for manipulating Xcode project files (.xcodeproj) programmatically using swift-argument-parser and the XcodeProj library.

## Key Technologies & Versions

- **Swift 6.0** - `@MainActor` isolation on commands, `Sendable` conformance on models
- **XcodeProj v9.7.2+** - Core manipulation library
- **PathKit v1.0.0+** - Uses `@preconcurrency` import until Swift 6 adoption
- **swift-argument-parser v1.5.0+** - `AsyncParsableCommand` pattern
- **macOS 10.15+** minimum

## Project Structure (Essential)

```
xcodeproj-cli/
├── Sources/xcodeproj-cli/          # Main implementation
├── Tests/xcodeproj-cliTests/       # Test suite
├── Package.swift                   # SPM configuration
├── build-universal.sh              # Universal binary build
└── .github/workflows/              # CI/CD automation
```

## Core Components (Quick Reference)

- **XcodeProjCLI** - Main ArgumentParser command structure
- **ProjectServiceFactory** - Service initialization from global options
- **GlobalOptions** - Shared CLI options (--project, --verbose, --dry-run)
- **ProjectServices** - Service coordination container
- **Services** - FileService, TargetService, GroupService, PackageService, BuildSettingsService
- **XcodeProjUtility** - Coordination layer
- **CacheManager** - Performance optimization
- **TransactionService** - Safe operations with backup/rollback
- **ValidationService** - Integrity checking

**📋 For detailed architecture information, see [ARCHITECTURE.md](./ARCHITECTURE.md)**

**See README.md for full command documentation.**


--- 


## Workflow Rules, Guardrails and Guidelines

### Foundational Rules and Guardrails
_Always fully read and understand this file before doing any work:_ @ai_docs/rules/CRITICAL-RULES-AND-GUARDRAILS.md


### Foundational Development Guidelines and Standards
Always fully read relevant guidelines below as needed, based on the type of work being done:
- _`ai_docs/guidelines/DEVELOPMENT-ARCHITECTURE-GUIDELINES.md`_ when doing development work (coding, architecture, etc.)
- _`ai_docs/guidelines/UX-UI-GUIDELINES.md`_ when doing UX/UI related work
- _`ai_docs/guidelines/WEB-DEV-GUIDELINES.md`_ when doing web development work


---


## Project Specific Development Philosophy

### Code Style
- Use Swift naming conventions (PascalCase for types, camelCase for methods)
- Prefer structs over classes for data models
- Prefer guard statements for early returns
- Use swift-format (v601.0.0+) for consistent formatting

### Error Handling
- Use custom `ProjectError` enum for domain-specific errors
- Provide actionable error messages with specific remediation steps
- Fail fast with clear error reporting
- Exit with meaningful codes (0 = success, 1 = error, specific codes for specific failures)

### Testing Philosophy
- Test suite uses real project manipulation (not mocks)
- Tests are organized by feature area
- Each test should be independent and restorable
- Always verify both positive and negative cases

### Documentation Guidelines
- Never document code that is self-explanatory
- Never write full API-level documentation for application code
- For complex or non-obvious code, add concise comments explaining the purpose and logic (but only when needed)

### **KNOWN DESIGN DECISIONS (Don't Second-Guess)**
- **Single `..` in paths is allowed** - This is intentional for parent directory access
- **XcodeProjUtility remains large** - Gradual migration planned, see ROADMAP.md
- **Binary-only distribution** - Swift script removed in v2.0.0, this is permanent
- **Homebrew as primary distribution** - Optimized for this installation method
- **No mocking in tests** - Real project manipulation is intentional for authenticity


## Testing and Code Analysis Guidelines

### Code Analysis and Style (Analysis, Linting and Formatting)

**IMPORTANT**: Only run formatting/linting commands after modifying Swift source or test code, and preferably only on the specific files that were changed.

```bash
# Swift code formatting on specific modified files (preferred)
swift-format format --in-place path/to/ModifiedFile.swift

# Swift code linting on specific modified files (preferred)
swift-format lint path/to/ModifiedFile.swift

# Format multiple specific files
swift-format format --in-place Sources/xcodeproj-cli/Services/FileService.swift Tests/xcodeproj-cliTests/FileOperationsTests.swift

# Only if you modified many files across a directory, format recursively
swift-format format --in-place --recursive Sources/xcodeproj-cli/Services
swift-format lint --recursive Sources/xcodeproj-cli/Services
```

### Running Tests

All tests use Swift Package Manager and are located in `Tests/xcodeproj-cliTests/`.

```bash
# Run all tests
swift test

# Run specific test suite
swift test --filter ValidationTests    # Read-only validation tests
swift test --filter FileOperationsTests # File manipulation tests
swift test --filter BuildAndTargetTests # Target and build tests
swift test --filter PackageTests        # Swift package tests
swift test --filter SecurityTests       # Security tests


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tolo/xcodeproj-cli](https://github.com/tolo/xcodeproj-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
