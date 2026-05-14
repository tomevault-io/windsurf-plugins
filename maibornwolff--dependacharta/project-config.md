---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

DependaCharta is a multi-language code analysis and visualization tool that consists of two main components:
- **Analysis**: Kotlin-based CLI tool using Tree-sitter parsers to analyze codebases
- **Visualization**: Angular/TypeScript web app using Cytoscape.js for interactive dependency graphs

## Build and Development Commands

### Quick Commands (using mise)

```bash
# Run analysis (Kotlin) unit tests
mise run test-analysis

# Run visualization (Angular) unit tests (headless)
mise run test-visualization

# Build the analysis tool
mise run build-analysis-jar

# Run analysis on a directory (builds first if needed)
mise run analyze <directory-to-analyze>

# Analyze a directory, prepare for visualization, and start server (builds first if needed)
mise run analyze-and-serve <directory-to-analyze>

# Clean up temporary analysis files from interrupted/crashed runs
mise run clean-temp

# Start visualization development server (installs deps automatically)
mise run dev-visualization
```

**Note**: The difference between `analyze` and `analyze-and-serve`:
- `mise run analyze <dir>` - Runs analysis and outputs to `output/analysis.cg.json`
- `mise run analyze-and-serve <dir>` - Runs analysis, outputs to `visualization/public/analysis/analyzed-project.cg.json`, and starts the frontend so you can view results directly.

### Analysis Component (Kotlin)

```bash
# Build the analysis tool
cd analysis
./gradlew fatJar

# Run tests
./gradlew test

# Run the analyzer
java -jar build/libs/dependacharta.jar -d <directory-to-analyze>
# Or use: bin/dependacharta.sh (Mac/Linux) or bin/dependacharta.bat (Windows)
```

### Visualization Component (Angular)

```bash
cd visualization

# Install dependencies
npm ci

# Development server (http://localhost:4200)
npm run start

# Run as Electron app
npm run start-electron

# Build
npm run build

# Run tests (opens Chrome, watches for changes - interactive dev mode)
npm run test

# Run tests headless (single run, for CI or quick verification)
npm run test -- --no-watch --browsers=ChromeHeadless

# Run E2E tests
npm run cypress:open  # Interactive
npm run cypress:run   # Headless

# Package for distribution
npm run package-win         # Windows
npm run package-mac-silicon # macOS ARM64
npm run package-mac-intel   # macOS x64
```

**Note for Claude Code**: When running Angular CLI commands directly (not via npm scripts), use `npx ng ...` to ensure the local binary is found in the sandboxed environment.

## Architecture Overview

### Analysis Flow
1. Tree-sitter parsers analyze source files without compilation
2. Creates dependency graph with cycle detection and levelization
3. Outputs `.cg.json` files containing the analysis results

For a detailed explanation of DependaCharta's domain concepts, see our [Domain Model Documentation](DOMAIN.md).

### Key Technologies
- **Analysis**: Kotlin, Tree-sitter, Gradle, Clikt CLI framework
- **Visualization**: Angular 20, TypeScript, Cytoscape.js, Electron
- **Supported Languages**: Java, C#, C++, TypeScript, JavaScript, PHP, Go, Python, Kotlin, Vue, Delphi

### Project Structure
```
analysis/
├── src/main/kotlin/        # Core analysis logic
│   ├── de/maibornwolff/    # Main package
│   └── parsing/            # Language-specific parsers
└── src/test/               # Test files

visualization/
├── src/app/                # Angular components
│   ├── codeCharta/         # Core visualization
│   └── model/              # Data models
└── src/assets/             # Static assets
```

### Important Patterns
- Language analyzers live in `analysis/src/main/kotlin/de/maibornwolff/dependacharta/pipeline/analysis/analyzers/<lang>/`
- Each analyzer implements `LanguageAnalyzer` and extracts packages, imports, declarations, and used types from AST
- **Java, Kotlin, and C#** delegate to the [TreeSitterExcavationSite](https://github.com/MaibornWolff/TreeSitterExcavationSite) (TSE) library via `TreeSitterDependencies.analyze()` and map TSE domain types to DC domain types. This is the target pattern for future language migrations.
- **Other languages** still use custom TreeSitter queries (TSQuery) directly. These can be migrated to TSE once it supports dependency analysis for the respective language.
- Angular components follow standard Angular patterns with services for data management
- Visualization uses reactive patterns with RxJS for state management

### Migrating a Language Analyzer to TSE

TSE provides dependency analysis via `TreeSitterDependencies.analyze(code, language)` returning a `DependencyResult` with `packagePath`, `imports`, and `declarations` (including `usedTypes`). When TSE adds support for a new language's dependency analysis, DC's analyzer for that language can be migrated.

**Steps:**
1. **In TSE**: Add dependency analysis support for the language (see TSE's `language-definitions.md` rule). Release a new TSE version.
2. **In DC**: Update the TSE dependency version in `analysis/build.gradle.kts`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MaibornWolff/DependaCharta](https://github.com/MaibornWolff/DependaCharta) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
