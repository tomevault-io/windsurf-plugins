---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

SwiftDependencyAudit is a Swift CLI tool that analyzes Swift Package.swift files, scans target source directories for import statements, and compares declared dependencies against actual usage. Built with Swift 6.1+, the tool provides parallel processing, colored output, and comprehensive dependency validation.

**Key Features:**
- Analyzes Package.swift files and extracts target dependencies
- Scans Swift source files for import statements using regex
- Identifies missing dependencies (imports without declarations)
- Identifies unused dependencies (declarations without imports)
- Configurable whitelist for system imports (Foundation, SwiftUI, etc.)
- Parallel processing with TaskGroup for performance
- Colored terminal output with JSON format option
- CLI interface built with Swift Argument Parser

## Development Commands

### Building
```bash
swift build
```

### Running
```bash
swift run swift-dependency-audit
```

### Testing
```bash
swift test
```

**Test Coverage:** Comprehensive test suite with 30 tests covering:
- ImportScanner regex parsing and whitelist functionality
- PackageParser Package.swift parsing logic
- DependencyAnalyzer target analysis and reporting
- WhitelistTests custom whitelist integration
- IntegrationTests end-to-end package analysis

## Architecture

This is a Swift Package Manager (SPM) project with library/executable split for testing:
- **Package.swift**: Defines package with library and executable targets
- **Sources/SwiftDependencyAuditLib/**: Core library components (public API)
  - **SwiftDependencyAudit.swift**: CLI interface with Swift Argument Parser
  - **PackageParser.swift**: Package.swift parsing logic
  - **ImportScanner.swift**: Source file import scanning with regex
  - **DependencyAnalyzer.swift**: Comparison and analysis logic
  - **ParallelProcessor.swift**: TaskGroup-based parallel processing
  - **ColorOutput.swift**: ANSI terminal color support
  - **Models.swift**: Data structures and types
- **Sources/SwiftDependencyAudit/**: Executable entry point
- **Tests/SwiftDependencyAuditTests/**: Comprehensive test suite

**Dependencies:**
- Swift Argument Parser for CLI interface
- Foundation for file system operations and JSON output
- Swift Standard Library for concurrency (TaskGroup) and regex parsing

## File Structure

```
SwiftDependencyAudit/
├── Package.swift                    # Package definition with library/executable targets
├── PROJECT_PLAN.md                  # Detailed project plan and implementation guide
├── CLAUDE.md                        # This file - development guidance
├── Sources/
│   ├── SwiftDependencyAuditLib/     # Core library (public API for testing)
│   │   ├── SwiftDependencyAudit.swift      # CLI interface
│   │   ├── PackageParser.swift             # Package.swift parsing
│   │   ├── ImportScanner.swift             # Import scanning with regex
│   │   ├── DependencyAnalyzer.swift        # Analysis logic
│   │   ├── ParallelProcessor.swift         # Parallel processing
│   │   ├── ColorOutput.swift               # Terminal colors
│   │   └── Models.swift                    # Data structures
│   └── SwiftDependencyAudit/        # Executable entry point
│       └── SwiftDependencyAudit.swift      # CLI main function
└── Tests/
    └── SwiftDependencyAuditTests/   # Comprehensive test suite
        ├── ImportScannerTests.swift        # Import parsing tests
        ├── PackageParserTests.swift        # Package parsing tests
        ├── DependencyAnalyzerTests.swift   # Analysis logic tests
        ├── WhitelistTests.swift            # Whitelist functionality
        └── IntegrationTests.swift          # End-to-end tests
```

## Development Notes

- Project uses Swift 6.1+ features and strict concurrency
- Uses Swift Argument Parser for CLI interface with async/await support
- Implements parallel processing with TaskGroup for performance
- Supports colored terminal output with ANSI escape sequences
- Provides JSON output format for automation
- Comprehensive test suite with 30 tests covering all functionality
- Library/executable split architecture for testability

## CLI Usage

```bash
# Analyze current directory
swift run swift-dependency-audit

# Analyze specific package
swift run swift-dependency-audit /path/to/package

# Use whitelist to ignore system frameworks
swift run swift-dependency-audit --whitelist "Foundation,SwiftUI,AppKit,UIKit"

# Options
swift run swift-dependency-audit --no-color --verbose --output-format json
swift run swift-dependency-audit --target MyTarget --exclude-tests
```

## Implementation Status

✅ **COMPLETED** - Full implementation with comprehensive testing!

All core features have been implemented and tested:
- ✅ Package.swift parsing with Swift 6.1 support
- ✅ Import statement scanning using Swift regex (including @testable imports)
- ✅ Dependency analysis and comparison logic
- ✅ Configurable whitelist for system imports
- ✅ Parallel processing with TaskGroup
- ✅ CLI interface with Swift Argument Parser
- ✅ Colored terminal output with ANSI support
- ✅ JSON output format for automation

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tonyarnold/swift-dependency-audit](https://github.com/tonyarnold/swift-dependency-audit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
