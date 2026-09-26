---
trigger: always_on
description: This file provides authoritative guidance to AI coding agents working with code in this repository.
---

# CLAUDE.md

This file provides authoritative guidance to AI coding agents working with code in this repository.

## Overview

Philips Roslyn Analyzers — custom Roslyn diagnostic analyzers for C# shipped as NuGet packages. The analyzers provide real-time compiler feedback and many include automatic code fixers. Open-sourced by Philips in 2020; all rules originate from real code review feedback.

## Build and Test Commands

Run from the repository root. All commands use `--configuration Release`.

```bash
# Build — generates NuGet packages in ./Packages/
dotnet build --configuration Release

# Run all tests (MSTest)
dotnet test --configuration Release --logger "trx;LogFileName=test-results.trx"

# Run a single test class
dotnet test --configuration Release --filter "FullyQualifiedName~AvoidThreadSleepTest"

# Run a single test method
dotnet test --configuration Release --filter "FullyQualifiedName~AvoidThreadSleepTest.BehindAlias"

# Verify code formatting
dotnet format style --verify-no-changes --no-restore --verbosity detailed

# Fix formatting violations
dotnet format style --no-restore
```

## Formatting Rules (zero tolerance — IDE0055 is severity error)

- Line endings: CRLF
- Indentation: tabs, size 4
- Encoding: UTF-8 with BOM for .cs files
- Braces: Allman style (new line before all braces)
- Parameters: camelCase
- See `.editorconfig` for the full set; `TreatWarningsAsErrors` is enabled in `Directory.Build.Common.props`

## Architecture

### Analyzer Class Hierarchy

All analyzers flow through a common base in `Philips.CodeAnalysis.Common`:

```
DiagnosticAnalyzer (Roslyn)
  └─ DiagnosticAnalyzerBase — sealed Initialize(), enables concurrent execution,
  │    creates Helper on CompilationStart, delegates to InitializeCompilation()
  │  └─ SingleDiagnosticAnalyzer — one DiagnosticId + one Rule
  │    └─ SingleDiagnosticAnalyzer<TNode, TSyntaxNodeAction> — auto-registers
  │         SyntaxNodeAction for the SyntaxKind inferred from TNode; handles
  │         generated-code filtering; instantiates TSyntaxNodeAction per node
  └─ SolutionAnalyzer — operates on the full Compilation (opt-in by default)
```

**Most analyzers** inherit `SingleDiagnosticAnalyzer<TNode, TSyntaxNodeAction>` and pair with a `SyntaxNodeAction<T>` subclass that implements `Analyze()`. The generic base auto-maps `TNode` to a `SyntaxKind` — override `GetSyntaxKind()` only if the default mapping doesn't fit.

### SyntaxNodeAction Pattern

The analysis logic lives in a `SyntaxNodeAction<T>` subclass (not in the analyzer). It receives `Context`, `Node`, `Rule`, `Helper`, and calls `ReportDiagnostic(location)`.

### Code Fix Hierarchy

```
CodeFixProvider (Roslyn)
  └─ SingleDiagnosticCodeFixProvider<TSyntax> — one fixable ID, BatchFixer FixAll,
  │    override ApplyFix()
  └─ SolutionCodeFixProvider<TSyntax> — operates across the solution
```

Code fix providers must be annotated with `[ExportCodeFixProvider(LanguageNames.CSharp, Name = nameof(...)), Shared]`.

### Helper System

`Helper` (inherits `CodeFixHelper`) is created per compilation and provides domain helpers accessed via properties:
- `ForAttributes` — attribute detection
- `ForNamespaces` — using/alias resolution
- `ForTypes`, `ForLiterals`, `ForModifiers`, `ForConstructors`, `ForTests`, `ForAssemblies`, `ForGeneratedCode`
- `ForAdditionalFiles` — reads .editorconfig settings and AdditionalFiles (exceptions lists)
- `ForAllowedSymbols` — allowlist support with wildcards
- `ForDocumentationOf(node)` — XML doc helpers

### DiagnosticId Enum

All diagnostic IDs live in `Philips.CodeAnalysis.Common/DiagnosticId.cs`. IDs use the `PH` prefix (e.g., `PH2160`). The enum's numeric value maps directly to the ID number.

### Project Layout

| Project | Purpose |
|---|---|
| `Common` | Base classes, helpers, DiagnosticId enum |
| `MaintainabilityAnalyzers` | Largest set — subcategories: Maintainability, Documentation, Naming, Readability, RuntimeFailure, Cardinality |
| `DuplicateCodeAnalyzer` | Duplicate code detection (uses Mono.Cecil) |
| `MoqAnalyzers` | Moq framework misuse detection |
| `MsTestAnalyzers` | MSTest framework rules |
| `SecurityAnalyzers` | Security rules (passwords, RSA padding, licensing) |
| `Test` | All unit tests (single project, mirrors analyzer structure) |
| `Benchmark` | BenchmarkDotNet performance tests |
| `AnalyzerPerformance` | Performance analysis tooling |

### Packaging and ILRepack

Each analyzer project targets `net8.0;netstandard2.0`. The `Directory.Build.Analyzer.props` configures ILRepack to merge `Common.dll` (and Mono.Cecil if present) into each analyzer DLL for the `netstandard2.0` target, so each NuGet package is self-contained. Packages output to `./Packages/`.

### Categories

Defined in `Common/Categories.cs`: Documentation, Maintainability, Naming, Readability, RuntimeFailure, Security, FunctionalProgramming, MsTest.

## Test Conventions

- Test framework: MSTest. All tests are in `Philips.CodeAnalysis.Test`.
- Test directory structure mirrors the analyzer project structure (e.g., `Test/Maintainability/Maintainability/`, `Test/Moq/`).
- Tests extend `DiagnosticVerifier` (analyzer-only) or `CodeFixVerifier` (analyzer + fixer).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [philips-software/roslyn-analyzers](https://github.com/philips-software/roslyn-analyzers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-25 -->
