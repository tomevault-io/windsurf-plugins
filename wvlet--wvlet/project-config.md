---
trigger: always_on
description: This file provides guidance to Gemini when working with code in this repository.
---

# GEMINI.md

This file provides guidance to Gemini when working with code in this repository.

## Project Overview

Wvlet is a cross-SQL flow-style query language for functional data modeling and interactive data exploration. It compiles .wv query files into SQL for various database engines (DuckDB, Trino, Hive). The project consists of a language compiler, runtime system, web UI, and multi-platform bindings.

## Key Development Commands

### Building and Installing
```bash
# Enter SBT shell
./sbt

# Install wvlet CLI command to ~/local/bin/wv
sbt:wvlet> cli/packInstall

# Build native library (requires clang, llvm, libgc)
sbt:wvlet> wvcLib/nativeLink

# Build standalone native compiler
sbt:wvlet> wvc/nativeLink
```

Note: The `wv` command is implemented in WvletREPLMain, while `wvlet` is implemented in WvletMain.

### Code Formatting

Ensure the code is formatted with `scalafmtAll` command for consistent code style. CI will check formatting on pull requests.

```bash
# Format code
./sbt scalafmtAll

# Check formatting
./sbt scalafmtCheck
```

### Testing
```bash
# Run all tests
./sbt test

# Run specific module tests
./sbt "runner/test"
./sbt "langJVM/test"

# Test specific module for Scala.js
./sbt "langJS/test"

# Compile all projects for individual platforms
./sbt "projectJVM/Test/compile"
./sbt "projectJS/Test/compile"
./sbt "projectNative/Test/compile"

# Run specific test class
./sbt "runner/testOnly *BasicSpec"

# Run a specific .wv spec file in BasicSpec
./sbt "runner/testOnly *BasicSpec -- spec:basic:hello.wv"

# Run a specific .wv spec files with wild card pattern
./sbt "runner/testOnly *BasicSpec -- spec:basic:query*.wv"

# Run test and stay in SBT shell
./sbt
sbt:wvlet> test
sbt:wvlet> runner/test
sbt:wvlet> testOnly *SpecRunner*

# Test native library with various languages
cd wvc-lib && make test
```

### UI Development
```bash
# Install JS dependencies once after clone
pnpm install

# Start main UI development server
pnpm run ui

# Start playground development server
pnpm run playground

# Build UI for production
pnpm run build-ui
pnpm run build-playground
```

### Documentation
```bash
# Start documentation server at localhost:3000
pnpm --filter website run start

# Build documentation
pnpm --filter website run build
```

## Architecture Overview

### Multi-Module SBT Structure
- **Core Language**: `wvlet-lang` (compiler, parser, analyzer), `wvlet-api` (cross-platform APIs)
- **Execution**: `wvlet-runner` (query executor with DB connectors), `wvlet-cli` (command-line interface)
- **Web Stack**: `wvlet-server` (HTTP API), `wvlet-ui*` (React/Scala.js components)
- **Multi-Platform**: JVM, JavaScript (Scala.js), Native (Scala Native) support
- **Language Bindings**: `wvc-lib` for C/C++/Rust integration

### Compiler Pipeline
1. **Parser**: Wvlet syntax (.wv files) → AST using custom parser combinators
2. **Analyzer**: Type resolution, symbol resolution, dependency analysis
3. **Code Generator**: AST → SQL for target database engines
4. **Runner**: SQL execution via database-specific connectors

### Database Connectors
- **DuckDB**: Default for testing and lightweight execution
- **Trino**: Production distributed query engine
- **Delta Lake**: Support for Delta table format

## Testing Framework

- Uses AirSpec testing framework https://wvlet.org/airspec/
- Test files end with `Test.scala` or `Spec.scala`
- Avoid using mock as it increases maintenance cost and creates brittle tests that break when internal implementation changes
- Ensure tests cover new functionality and bug fixes with good test coverage
- Test names should be concise and descriptive, written in plain English
    - Good: `"should parse JSON with nested objects"`, `"should handle connection timeout gracefully"`
    - Avoid: `"testParseJSON"`, `"test1"`, `"shouldWork"`


### Spec-Driven Testing
The project uses a unique **spec-driven testing approach** where `.wv` files in `spec/` directory serve as executable test cases:

- `spec/basic/`: Core functionality tests (149+ .wv files)
- `spec/tpch/`: TPC-H benchmark queries
- `spec/neg/`: Negative test cases (expect compilation/execution errors)
- `spec/cdp_*/`: Customer Data Platform behavior tests

- **Embedded Assertions**: `.wv` files contain `test` statements for validation
- **SpecRunner**: Core engine that compiles and executes .wv files as test cases


### Test Assertions in .wv Files
```wv
from 'data.json'
test _.size should be 10
test _.columns should contain 'user_id'
test _.output should be """
| user_id | name     |
|---------|----------|
| 1       | Alice    |
"""
```

## Development Patterns

### File Organization
- `.wv` files: Wvlet query language source files
- Scala sources: Follow standard Maven/SBT directory structure
- Cross-platform code: Use `%%%` for multi-platform dependencies
- **Platform specific code needs to be placed in .jvm/src/main/scala, .js/src/main/scala, .native/src/main/scala folders**

### Code Style
- **Scala 3**: Latest Scala version (check `SCALA_VERSION` file). No Scala 2 support needed.
- For cross-platform projects, use .jvm, .js, and .native folders for platform-specific code
- Omit `new` for object instantiation (e.g., `StringBuilder()` instead of `new StringBuilder()`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wvlet/wvlet](https://github.com/wvlet/wvlet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->
