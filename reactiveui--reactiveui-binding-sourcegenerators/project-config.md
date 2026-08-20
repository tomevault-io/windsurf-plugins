---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Zero Tolerance Policy

- **NEVER abandon work halfway through** - if something gets difficult, push through it
- **NEVER use `git stash`** to hide incomplete work - fix the problem directly
- **NEVER give up because a task is complex** - break it down and keep going
- If a tool call is rejected, adapt your approach immediately and continue

## Build & Test Commands

This project uses **Microsoft Testing Platform (MTP)** with the **TUnit** testing framework. Test commands differ significantly from traditional VSTest.

See: https://learn.microsoft.com/en-us/dotnet/core/tools/dotnet-test?tabs=dotnet-test-with-mtp

### Prerequisites

```powershell
# Check .NET installation (.NET 8.0, 9.0, and 10.0 required)
dotnet --info

# Restore NuGet packages
cd src
dotnet restore ReactiveUI.Binding.SourceGenerators.slnx
```

**Note:** This project uses the modern `.slnx` (XML-based solution file) format instead of the legacy `.sln` format.

### Build Commands

**CRITICAL:** The working folder must be `./src` folder. These commands won't function properly without the correct working folder.

```powershell
# Build the solution
dotnet build ReactiveUI.Binding.SourceGenerators.slnx -c Release

# Build with warnings as errors (includes StyleCop violations)
dotnet build ReactiveUI.Binding.SourceGenerators.slnx -c Release -warnaserror

# Clean the solution
dotnet clean ReactiveUI.Binding.SourceGenerators.slnx
```

### Test Commands (Microsoft Testing Platform)

**CRITICAL:** This repository uses MTP configured in `testconfig.json`. All TUnit-specific arguments must be passed after `--`:

The working folder must be `./src` folder.

**IMPORTANT:**
- Do NOT use `--no-build` flag when running tests. Always build before testing to ensure all code changes are compiled.
- Use `--output Detailed` to see Console.WriteLine output from tests (place BEFORE any `--` separator).

```powershell
# Run all tests in the solution
dotnet test --solution ReactiveUI.Binding.SourceGenerators.slnx -c Release

# Run all tests in a specific project
dotnet test --project tests/ReactiveUI.Binding.Analyzer.Tests/ReactiveUI.Binding.Analyzer.Tests.csproj -c Release
dotnet test --project tests/ReactiveUI.Binding.SourceGenerators.Tests/ReactiveUI.Binding.SourceGenerators.Tests.csproj -c Release
dotnet test --project tests/ReactiveUI.Binding.Tests/ReactiveUI.Binding.Tests.csproj -c Release

# Run a single test method using treenode-filter
dotnet test --project tests/ReactiveUI.Binding.SourceGenerators.Tests/ReactiveUI.Binding.SourceGenerators.Tests.csproj -- --treenode-filter "/*/*/*/MyTestMethod"

# Run all tests in a specific class
dotnet test --project tests/ReactiveUI.Binding.SourceGenerators.Tests/ReactiveUI.Binding.SourceGenerators.Tests.csproj -- --treenode-filter "/*/*/WhenChangedGeneratorTests/*"

# Run tests with code coverage
dotnet test --solution ReactiveUI.Binding.SourceGenerators.slnx -- --coverage --coverage-output-format cobertura
```

### TUnit Treenode-Filter Syntax

The `--treenode-filter` follows the pattern: `/{AssemblyName}/{Namespace}/{ClassName}/{TestMethodName}`

- Single test: `--treenode-filter "/*/*/*/MyTestMethod"`
- All tests in class: `--treenode-filter "/*/*/MyClassName/*"`
- Use single asterisks (`*`) to match segments.

### Key Configuration Files

- `src/ReactiveUI.Binding.SourceGenerators.slnx` - Modern XML-based solution file
- `src/testconfig.json` - Configures test execution and code coverage
- `src/Directory.Build.props` - Common build properties, package metadata
- `src/Directory.Packages.props` - Central package management
- `src/Directory.Build.targets` - Build targets

### Snapshot Testing with Verify

- Generator tests use **Verify.SourceGenerators** for snapshot testing
- Snapshots stored as `*.verified.cs` files alongside test classes
- To accept new/changed snapshots:
  1. Enable `VerifierSettings.AutoVerify()` in `AssemblySetup.cs`
  2. Run tests to accept all snapshots
  3. Disable `VerifierSettings.AutoVerify()` after accepting
  4. Re-run tests to confirm they pass without AutoVerify

### Generator Test Language Versions (Critical)

Generator tests use a **two-tier language version** strategy to verify generated output compiles under C# 7.3 (the minimum supported version for consumer projects):

- **Default: C# 7.3** — `TestHelper.CreateCompilation()` and `RunGenerator()` default to `LanguageVersion.CSharp7_3`. This ensures generated output contains no C# 8+ syntax (no nullable reference type annotations, no `static` lambdas, no `#nullable enable`).
- **CallerArgumentExpression tests: explicit C# 10** — Tests that verify `CallerArgumentExpression`-based dispatch (the primary dispatch mechanism for C# 10+ projects) must pass `LanguageVersion.CSharp10` explicitly. These are the majority of snapshot tests.
- **CallerFilePath fallback tests: explicit C# 7.3** — Tests that verify `CallerFilePath + CallerLineNumber` dispatch (the fallback for pre-C# 10 projects) pass `LanguageVersion.CSharp7_3` explicitly to document intent, even though it matches the default.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [reactiveui/ReactiveUI.Binding.SourceGenerators](https://github.com/reactiveui/ReactiveUI.Binding.SourceGenerators) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-20 -->
