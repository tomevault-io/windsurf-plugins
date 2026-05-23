---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with this repository.

## Project Overview

**Z3Wrap** is a production-ready C# wrapper for Microsoft's Z3 theorem prover featuring unlimited precision arithmetic, type-safe API design, and natural mathematical syntax.

**Technology**: .NET 9.0 library with comprehensive test coverage (run `make test` to see current stats).

## Development Workflow

**CRITICAL**: This project uses a comprehensive Makefile for ALL development tasks. NEVER use direct `dotnet` commands.

### Essential Commands
```bash
make help         # Show all available commands
make build        # Build library (includes restore)
make test         # Run all tests (see count in output)
make coverage     # Generate coverage report and open in browser
make format       # Format code (CSharpier) - REQUIRED before commits
make lint         # Check formatting (used in CI)
make ci           # Full CI pipeline (lint, build, test, coverage)
make clean        # Clean all build artifacts
```

### Why Makefile-First?
- **Consistency**: Same commands across all platforms and environments
- **Dependencies**: Automatic prerequisite handling (restore → build → test)
- **CI Integration**: Identical commands locally and in GitHub Actions
- **Tool Management**: Handles installation checks and provides helpful error messages

**Rule**: Always use `make [command]` instead of `dotnet [command]` to ensure consistent behavior.

## Project Structure Discovery

**Don't assume file locations or counts - always verify using tools:**
- Use `Glob` to find files by pattern (e.g., `**/*.cs`, `**/Z3*.cs`)
- Use `Grep` to search for code patterns
- Use `Read` to examine specific files
- Run `make test` to see current test count
- Run `make coverage` to see current coverage percentage

**Organization Pattern**:
```
Z3Wrap.sln                    # Solution with library + tests
├── Z3Wrap/                   # Main library
│   ├── Values/                  # Bv<TSize>, Real value types
│   ├── Expressions/             # Expression types organized by category
│   │   ├── Arrays/              # ArrayExpr<TIndex, TValue>
│   │   ├── BitVectors/          # BvExpr<TSize> with Size8/16/32/64
│   │   ├── Functions/           # FuncDecl for uninterpreted functions
│   │   ├── Logic/               # BoolExpr
│   │   ├── Numerics/            # IntExpr, RealExpr
│   │   ├── Strings/             # StringExpr, CharExpr
│   │   └── Quantifiers/         # ForAll, Exists
│   ├── Core/                    # Z3Context, Z3Solver, Z3Model, Z3Optimizer
│   │   ├── Interop/             # P/Invoke bindings, native library loading
│   │   └── Z3Library.cs         # Complete Z3 C API wrapper
│   └── *.csproj                 # Project configuration
├── Z3Wrap.Tests/             # NUnit tests organized by category
│   ├── Core/                    # Context, Solver, Model, Optimizer tests
│   ├── Expressions/             # Expression tests organized by type
│   ├── Values/                  # Bv and Real value type tests
│   └── ReadmeExamplesTests.cs   # Validates all README examples
├── docs/                     # Extended documentation
│   └── examples/                # Real-world use case examples
│       └── StringTheory.md      # String constraint solving examples
├── Makefile                  # Development workflow automation
├── CLAUDE.md                 # AI assistant guidance (this file)
├── PLAN.md                   # Project roadmap and status
└── .github/workflows/ci.yml  # GitHub Actions CI pipeline
```

## Key Features

- **Unlimited Precision**: BigInteger integers, exact rational arithmetic via Real class
- **Type Safety**: Strongly typed expressions, generic constraints, compile-time checking
- **Natural Syntax**: Mathematical operators (`x + y == 10`) via scoped context pattern
- **Complete Z3 Coverage**: Booleans, Integers, Reals, BitVectors, Arrays, Quantifiers, Uninterpreted Functions, Optimization
- **Memory Safety**: Reference-counted contexts, automatic cleanup, no resource leaks
- **Cross-Platform**: Auto-discovery of Z3 native library on Windows, macOS, Linux

## Testing Guidelines

**Coverage Requirement**: Maintain ≥90% line coverage (enforced by CI - run `make coverage` to verify)

**Test Execution**: Always use `make test` (never `dotnet test`)

**Naming**: `MethodName_Scenario_ExpectedResult` pattern

**Organization**: Hierarchical structure by expression type and functionality (use Glob to discover)

### Expression Operation Testing Principles

**CRITICAL**: All expression operations must test ALL syntax variants comprehensively.

**Complete Syntax Variant Coverage** (8 variants for binary operations):
```csharp
[Test]
public void Add_TwoValues_ComputesCorrectResult()
{
    using var context = new Z3Context();
    using var scope = context.SetUp();
    using var solver = context.CreateSolver();

    var a = context.Int(10);
    var b = context.Int(32);

    // Test ALL 8 syntax variants
    var result = a + b;                              // 1. Operator
    var resultViaIntLeft = 10 + b;                   // 2. Literal left
    var resultViaIntRight = a + 32;                  // 3. Literal right

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [spaceorc/Z3Wrap](https://github.com/spaceorc/Z3Wrap) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-23 -->
