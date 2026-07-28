---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

CodeCharta is a code visualization tool that generates 3D treemap visualizations of codebases. It consists of two main components:

- **Analysis (Kotlin/Java)**: CLI tool (`ccsh`) that parses source code, imports metrics from external tools, and generates `.cc.json` files
- **Visualization (TypeScript/Angular)**: Web and desktop application that renders `.cc.json` files as interactive 3D treemaps using Three.js

## Requirements

- Java >= 11, <= 21
- Node >= 20
- Git (with bash utilities for Windows)

## Common Development Commands

### Root Directory

```bash
# Install root dependencies (Husky, BiomeJS for formatting)
npm i

# Format all files
npm run format

# Check formatting
npm run format:check
```

### Analysis (Kotlin/Gradle)

```bash
cd analysis

# Build the project (creates distribution in build/)
./gradlew build

# Assemble without running tests (faster for frequent builds)
./gradlew assemble

# Install distribution locally for testing
./gradlew installDist

# Run all tests
./gradlew test

# Run integration tests (requires bash on Windows, timeout on macOS)
./gradlew integrationTest

# Check code style
./gradlew ktlintCheck

# Auto-format code
./gradlew ktlintFormat

# Run a single test class
./gradlew test --tests "ClassName"

# Run tests in a specific package
./gradlew test --tests "package.name.*"

# Run Sonar analysis
./gradlew sonar

# Use installed ccsh (after installDist)
./build/install/codecharta-analysis/bin/ccsh
```

**IMPORTANT**: All `./gradlew` commands must be run from the `analysis/` directory. If you are in a subdirectory, you must either:
1. Change to the analysis directory first: `cd /path/to/analysis && ./gradlew test`
2. OR use the full relative path: `cd /path/to/analysis && ./gradlew :subproject:test`

Do NOT run `./gradlew` from subdirectories as it will fail with "no such file or directory".

**Note**: On Windows, add Git's `sh.exe` (typically `C:\path-to-git\Git\bin`) to PATH for integration tests. On macOS, install `timeout` via `brew install coreutils`.

### Visualization (TypeScript/Angular)

```bash
cd visualization

# Install dependencies
npm i

# Start development server (web version)
npm run dev

# Build for production
npm run build

# Run all unit tests
npm test

# Run tests in watch mode
npm run test:auto

# Run tests without coverage
npm run test:autoNoCoverage

# Update snapshots
npm run test:updateSnaps

# Run e2e tests
npm run e2e

# Run e2e tests in CI mode (sequential)
npm run e2e:ci

# Run e2e tests in watch mode
npm run e2e:auto

# Start desktop client (requires build first)
npm run start

# Package desktop app for distribution
npm run package

# Generate JSON schema from TypeScript types
npm run schema:generate
```

## Architecture Overview

### Analysis Architecture

**Design Pattern**: Pipes and Filters architecture with plugin-based analysers

**Main Entry Point**: `analysis/ccsh/src/main/kotlin/de/maibornwolff/codecharta/ccsh/Ccsh.kt`
- Uses Picocli for CLI command structure
- Supports interactive mode, direct execution, and pipe chaining
- Registers all analysers (parsers, importers, filters, tools)

**Core Components**:

1. **AnalyserInterface**: Base interface for all analysers with methods:
   - `call()`: Execute the analyser
   - `isApplicable()`: Check if applicable to input
   - `getDialog()`: Interactive configuration UI

2. **Parsers** (`analysis/analysers/parsers/`): Extract metrics from source code
   - `GitLogParser`: Git commit history metrics
   - `SVNLogParser`: SVN log metrics
   - `RawTextParser`: Generic text metrics
   - `UnifiedParser`: Multi-language source code parser (modern replacement for SourceCodeParser)

3. **Importers** (`analysis/analysers/importers/`): Import from external tools
   - `SonarImporter`: SonarQube metrics
   - `CoverageImporter`: Code coverage data
   - `CSVImporter`: Generic CSV metrics
   - `CodeMaatImporter`: CodeMaat output
   - `TokeiImporter`: Tokei language statistics
   - `SourceMonitorImporter`: SourceMonitor data

4. **Filters** (`analysis/analysers/filters/`): Transform cc.json files
   - `MergeFilter`: Combine multiple cc.json files (supports recursive/leaf strategies, MIMO mode)
   - `EdgeFilter`: Aggregate edge attributes into nodes
   - `StructureModifier`: Modify file/folder tree structure

5. **Tools** (`analysis/analysers/tools/`):
   - `ValidationTool`: Validate cc.json schema compliance
   - `InspectionTool`: Display cc.json file information

The UnifiedParser is the most actively developed component:
- Uses TreeSitter for parsing multiple languages
- Metric calculators extend `MetricPerFileCalc` or `MetricPerFunctionCalc`
- Language support defined in `metricnodetypes/` with node type mappings to language constructs
- Each metric calculator implements `calculateMetricForNode(params: CalculationContext)`

**Data Model** (`analysis/model/`):
- `Project`: Root container (API version 1.5)
  - `rootNode`: Single root Node
  - `edges`: List of Edge connections
  - `attributeTypes`: Metric type definitions
  - `attributeDescriptors`: Metric metadata
  - `blacklist`: Excluded items

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MaibornWolff/codecharta](https://github.com/MaibornWolff/codecharta) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
