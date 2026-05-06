---
trigger: always_on
description: XLKit is a modern Swift library for creating and manipulating Excel (.xlsx) files on macOS and iOS. Built with Swift 6.0, targeting macOS 12+ and iOS 15+, using modular SPM architecture. iOS support is available and tested in CI/CD, with platform-specific code handling for iOS compatibility.
---

# XLKit - Cursor Rules for AI Agents

## Project Overview
XLKit is a modern Swift library for creating and manipulating Excel (.xlsx) files on macOS and iOS. Built with Swift 6.0, targeting macOS 12+ and iOS 15+, using modular SPM architecture. iOS support is available and tested in CI/CD, with platform-specific code handling for iOS compatibility.

## Architecture & Module Structure

### Core Modules
- XLKitCore: Core types, data structures, utilities (Workbook, Sheet, Cell, etc.)
- XLKitFormatters: CSV/TSV import/export functionality
- XLKitImages: Image processing and embedding utilities
- XLKitXLSX: XLSX file generation engine
- XLKit: Main API that re-exports all submodules

### Module Dependencies
```
XLKit (main API)
├── XLKitCore (core types & utilities)
├── XLKitFormatters (CSV/TSV import/export)
├── XLKitImages (image processing & embedding)
└── XLKitXLSX (XLSX generation engine)

XLKitFormatters
├── XLKitCore
└── TextFile (swift-textfile)

XLKitImages
└── XLKitCore

XLKitXLSX
├── XLKitCore
├── XLKitFormatters
└── XLKitImages
```

### Executable Target
```
XLKitTestRunner (executable)
└── XLKit
```

### XLKitTestRunner Overview

Purpose: Modular test runner for generating Excel files for testing and demonstration purposes.

Structure:
```
Sources/XLKitTestRunner/
├── main.swift                    # Entry point with command-line interface
├── ExcelGenerators.swift         # Excel generation functions
├── ImageEmbedGenerators.swift    # Image embedding tests
├── Templates/                    # Template files for new tests
│   └── TestGeneratorTemplate.swift
└── README.md                     # Documentation
```

Usage:
```bash
# Run specific test types
swift run XLKitTestRunner no-embeds
swift run XLKitTestRunner embed
swift run XLKitTestRunner comprehensive
swift run XLKitTestRunner help

# Show help
swift run XLKitTestRunner help
```

Available Test Types:
- `no-embeds` / `no-images` - Generate Excel from CSV without images
- `embed` / `with-embeds` / `with-images` - Generate Excel with embedded images from CSV data
- `comprehensive` / `demo` - Comprehensive API demonstration with all features
- `security-demo` / `security` - Demonstrate file path security restrictions
- `ios-test` / `ios` - Test iOS file system compatibility and platform-specific features
- `number-formats` / `formats` - Test number formatting (currency, percentage, custom formats)
- `help` / `-h` / `--help` - Show available commands

Test Features:
- Security Integration: All tests include security logging and validation
- CoreXLSX Validation: Generated files are validated for Excel compliance
- Aspect Ratio Testing: Image embedding tests all 17 professional aspect ratios
- Performance Testing: Large dataset handling and memory optimization
- Error Handling: Comprehensive error testing and edge case coverage
- Platform Testing: iOS compatibility validation and sandbox restrictions testing

Adding New Tests:
1. Copy template: `cp Sources/XLKitTestRunner/Templates/TestGeneratorTemplate.swift Sources/XLKitTestRunner/YourTestName.swift`
2. Modify function name and logic
3. Register in main.swift switch statement
4. Update help text
5. Create GitHub Actions workflow if needed

Naming Conventions:
- Function names: camelCase (e.g., `generateExcelWithImages()`)
- Test types: kebab-case (e.g., `with-images`, `csv-import`)
- File names: PascalCase (e.g., `ExcelGenerators.swift`)

Output Structure:
```
Test-Workflows/
├── Embed-Test.xlsx          # From no-embeds test
├── Embed-Test-Embed.xlsx    # From embed test (with images)
├── Comprehensive-Demo.xlsx  # From comprehensive test
├── Number-Format-Test.xlsx  # From number-formats test
└── [Your-Test].xlsx         # From custom tests

Root Directory:
├── iOS-Example.xlsx         # From ios-test (iOS compatibility)
└── [Other-Test].xlsx        # From other platform-specific tests
```

Security Features in Tests:
- Rate Limiting: Prevents test abuse and resource exhaustion
- Security Logging: All test operations are logged for audit trails
- Input Validation: All test inputs are validated for security
- File Quarantine: Suspicious test files are automatically quarantined
- Checksum Verification: Optional file integrity verification (disabled by default)

## File Organization & Paths

### Complete Directory Structure

```
XLKit/
├── AGENT.MD                     # AI agent development guide
├── .cursorrules                 # Cursor rules for AI agents
├── CHANGELOG.md                 # Version history and changes
├── LICENSE                      # MIT license
├── Package.swift                # Swift Package Manager configuration
├── Package.resolved             # Locked dependency versions
├── README.md                    # Main documentation
├── SECURITY.md                  # Security policy
├── .gitignore                   # Git ignore patterns
├── .swift-format                # Swift formatting configuration
├── Assets/                      # Project assets
│   └── XLKit_Icon.png          # Project icon

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TheAcharya/XLKit](https://github.com/TheAcharya/XLKit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
