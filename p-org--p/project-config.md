---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

P is a state machine based programming language for formally modeling and specifying complex distributed systems. The P framework allows programmers to model their system design as a collection of communicating state machines and provides automated reasoning backends (model checking, symbolic execution) to verify correctness specifications.

## Project Architecture

The P framework consists of five main components organized in `Src/`:

- **PCompiler** (`Src/PCompiler/`): The P language compiler that parses P programs and generates target code
  - `CompilerCore/`: Core compilation logic and AST handling
  - `PCommandLine/`: Command-line interface for the P compiler

- **PChecker** (`Src/PChecker/`): Model checker and systematic testing engine for P programs
  - `CheckerCore/`: Core model checking and systematic testing logic
  - `CoverageReportMerger/`: Tool for merging coverage reports

- **PEx** (`Src/PEx/`): Java-based execution engine that provides symbolic execution capabilities

- **PObserve** (`Src/PObserve/`): Runtime monitoring that validates service logs against P specification monitors, bridging design-time verification with production behavior
  - `PObserve/`: Core runtime monitor that replays log events through P spec machines
  - `PObserveCommons/`: Shared utilities/types between the monitor and consumers
  - `PObserveJavaUnitTest/`: Java unit tests for the monitor
  - `PObserveRegressionTesting/`: End-to-end regression suite that exercises monitors against recorded log corpora
  - `Examples/`: Reference integrations showing how to wire service logs into a P monitor

- **PeasyAI** (`Src/PeasyAI/`): AI-powered system for P language development assistance
  - `src/core/`: Core logic including different interaction modes
  - `src/ui/`: User interface components (Streamlit web app, CLI, MCP server)
  - `src/utils/`: Utilities for code generation, compilation, and P language analysis
  - `resources/`: Context files, examples, and instruction templates for AI interactions

The project follows a multi-language approach with the compiler written in C# (.NET), the execution engine in Java with Maven build system, and the AI assistant system in Python.

## Common Development Commands

### Building the Project

```bash
# Build the entire project (recommended)
./Bld/build.sh

# Build with specific configuration
./Bld/build.sh --config Debug
./Bld/build.sh --config Release

# Build with verbose output
./Bld/build.sh --verbose

# Install P as a global dotnet tool after building
./Bld/build.sh --install
```

Alternative using dotnet CLI:
```bash
# Build all projects
dotnet build

# Build with specific configuration
dotnet build --configuration Release
dotnet build --configuration Debug
```

### Running Tests

```bash
# Run all tests
dotnet test

# Run tests with specific configuration
dotnet test --configuration Release

# Run specific test project
dotnet test Tst/UnitTests/UnitTests.csproj

# Run a single test by name
dotnet test --filter "FullyQualifiedName~TestClassName.TestMethodName"

# Run tests in a specific category
dotnet test --filter "Category=Unit"

# Run tests with detailed output
dotnet test --logger "console;verbosity=detailed"
```

### Working with P Programs

```bash
# Compile a P program (requires P tool to be installed)
p compile

# Run model checking on a P program
p check

# Compile with PEx backend
p compile --mode pex

# Run model checking with PEx backend
p check --mode pex

# By default, `p compile` reports ALL type errors in one pass.
# Use --strict-errors / -se to restore the legacy abort-on-first behavior.
p compile --strict-errors
```

## Multi-Error Type Checking (Compiler)

The C# compiler under `Src/PCompiler/CompilerCore/` reports all type errors
in one pass by default (P 3.0+). Diagnostics flow through
`IDiagnosticCollector` and are flushed (sorted by source location) at end of
compilation. Users opt out via `--strict-errors` (`-se`), which restores the
legacy abort-on-first behavior.

### Architecture

- **`IDiagnosticCollector` / `DefaultDiagnosticCollector`** — strict mode
  rethrows immediately; collecting mode (default) appends to a list. The
  collector is flushed via `Compiler.FlushCollectedDiagnostics` at end of
  compilation.
- **`ErrorType` (singleton sentinel) / `ErrorExpr`** — substituted for
  failed-to-type-check expressions. `ErrorType` claims compatibility with
  every other type so downstream compatibility checks cascade-suppress.
- **`PLanguageType.IsSameTypeAs`** has a symmetric short-circuit when either
  side is `ErrorType`.
- **`TypeCheckingUtils.CheckAssignable`** is the cascade-aware compatibility
  check helper; visitors should route compatibility checks through this
  helper rather than calling `IsAssignableFrom` directly.
- **`Analyzer.TolerantStep`** is the per-pass-item analog: it wraps each
  iteration in `try { body() } catch (TranslationException) when (collecting)`
  so one bad machine/function doesn't clobber siblings' diagnostics.

### When adding a new throw site in a visitor

Follow the convention in `ExprVisitor.cs`'s class doc:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [p-org/P](https://github.com/p-org/P) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
