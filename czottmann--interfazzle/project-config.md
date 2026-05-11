---
trigger: always_on
description: Generates interface-style Markdown similar to Xcode's generated interfaces:
---

# AGENTS.md

This file provides guidance to LLM agents when working with code in this repository.

## Project Overview

interfazzle is a Swift package that generates Markdown documentation for Swift package public interfaces from symbol graphs. It's designed as a replacement for SourceDocs, avoiding crashes when packages include dependencies with pre-built binaries.

**Key characteristics:**

- Proper Swift package with library and executable targets
- Uses SwiftCLI framework for command-line interface
- Modular architecture with clear separation of concerns
- Comprehensive unit tests using Swift Testing framework
- Targeted at package maintainers and LLM-friendly documentation

## Package Structure

The package follows Swift Package Manager conventions:

```
interfazzle/
├── Package.swift              # Package manifest
├── Sources/
│   ├── Interfazzle/          # Library module (public API)
│   │   ├── Models/           # Data structures (Config, SymbolGraph, PackageDescription)
│   │   ├── Core/             # Workflow components (validation, building, extraction)
│   │   └── Generation/       # Documentation generation (formatting, sorting, output)
│   └── InterfazzleCLI/       # Executable module
│       ├── InterfazzleCLI.swift  # @main entry point
│       └── Commands/         # CLI commands (Generate, Build, Validate)
└── Tests/
    └── InterfazzleTests/     # Unit tests
```

The package in the `Demo/` directory is only there for demonstration purposes. It's not part of the Interfazzle package. Leave it alone.

## Core Modules

### Interfazzle Library

The core library provides all documentation generation functionality:

**Models/** - Data structures

- `Config.swift` - Configuration with command-line options
- `SymbolGraph.swift` - Codable types for Swift compiler symbol graphs
- `PackageDescription.swift` - Package metadata from `swift package describe`

**Core/** - Workflow components

- `PackageValidator.swift` - Validates Package.swift existence
- `SymbolGraphBuilder.swift` - Orchestrates Swift compiler to generate symbol graphs
- `ModuleExtractor.swift` - Extracts public module names from Package.swift
- `PackageInfoLoader.swift` - Loads target path information for README integration
- `PackageInfoProvider.swift` - Centralized provider for Swift package information with caching to eliminate duplicate process spawns

**Generation/** - Documentation generation

- `DocumentationGenerator.swift` - Main orchestrator for doc generation
- `SymbolSorter.swift` - Sorts symbols by dependency and type hierarchy
- `DeclarationFormatter.swift` - Formats Swift declaration fragments
- `MarkdownFormatter.swift` - Generates final Markdown output

### InterfazzleCLI Executable

Command-line interface using SwiftCLI framework:

- `InterfazzleCLI.swift` - Entry point with CLI initialization
- `GenerateCommand.swift` - Main workflow (build + generate docs)
- `BuildCommand.swift` - Build symbol graphs only
- `ValidateCommand.swift` - Validate Package.swift existence

## Common Development Commands

### Building and Running

```bash
# Build the package
swift build

# Run interfazzle
swift run interfazzle generate

# Run with flags
swift run interfazzle generate --verbose --be-lenient
swift run interfazzle generate --generate-only

# Show help
swift run interfazzle --help
swift run interfazzle generate --help

# Other commands
swift run interfazzle build
swift run interfazzle validate
```

### Installing Locally

```bash
# Build release version
swift build -c release

# Copy to PATH
cp .build/release/interfazzle /usr/local/bin/
```

### Testing

```bash
# Run all tests
swift test

# Build and run tests
swift test --parallel
```

### Development/Debugging Commands

```bash
# Manual symbol graph generation
swift build -Xswiftc -emit-symbol-graph -Xswiftc -emit-symbol-graph-dir -Xswiftc .build/symbol-graphs

# Package analysis
swift package describe --type json

# Clean build artifacts
swift package clean
rm -rf .build
```

## CLI Commands

### generate

Generate complete documentation (build + convert):

```bash
interfazzle generate [options]
```

**Flags:**

- `--generate-only` - Skip build phase, use existing symbol graphs
- `-v, --verbose` - Show full build output
- `--be-lenient` - Continue with existing graphs if build fails
- `--include-reexported` - Include re-exported symbols from dependencies
- `--symbol-graphs-dir <dir>` - Symbol graph directory (default: `.build/symbol-graphs`)
- `--output-dir <dir>` - Output directory (default: `docs`)
- `--modules <list>` - Comma-separated module list (default: all public products)

### build

Build symbol graphs without generating documentation:

```bash
interfazzle build [options]
```

**Flags:**

- `-v, --verbose` - Show full build output
- `--symbol-graphs-dir <dir>` - Symbol graph directory (default: `.build/symbol-graphs`)

### validate

Verify Package.swift exists in current directory:

```bash
interfazzle validate
```

## Code Organization Patterns

**Swift 6 Compliance:**

- Uses `Sendable` conformance for concurrency safety
- `@preconcurrency` imports for SwiftCLI
- Modern `async`/`await` patterns where appropriate

**Error Handling:**

- Uses Swift Error protocol with LocalizedError

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [czottmann/interfazzle](https://github.com/czottmann/interfazzle) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
