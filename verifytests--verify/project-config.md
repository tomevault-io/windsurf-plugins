---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Verify is a snapshot testing framework for .NET that simplifies assertion of complex data models and documents. It serializes test results to `.verified.*` files and compares them against actual results on subsequent runs.

**Solution file**: `src\Verify.slnx` (uses modern .slnx format)

## Build & Test Commands

**Always pass `src/Verify.slnx` explicitly** to `dotnet build`. There are multiple `.slnx` files in `src/` and omitting the solution file causes ambiguity errors.

```bash
# Build the whole solution
dotnet build src/Verify.slnx
```

### Running the full test suite

`global.json` pins the test runner to Microsoft.Testing.Platform (MTP). Because `Verify.Fixie.Tests` and `Verify.Expecto.FSharpTests` still use the VSTest runner, **`dotnet test src/Verify.slnx` does not work** — it aborts with a mixed-runner error ("All projects must use that test runner") before running anything. There is no single command that runs every test project.

Instead, build once and run each test project individually. **`src/appveyor.yml` (`build_script`) is the source of truth** for the complete, ordered list and how each project is invoked. The commands differ by runner:

```bash
# Build once in the target configuration, then run with --no-build --no-restore
dotnet build src/Verify.slnx -c Release

# Most projects (xUnit v3, NUnit, MSTest) run via `dotnet test`
dotnet test src/Verify.Tests -c Release --no-build --no-restore
dotnet test src/Verify.NUnit.Tests -c Release --no-build --no-restore

# Expecto (F#) and TUnit projects are console executables — run via `dotnet run`
dotnet run --project src/Verify.Expecto.Tests -c Release --no-build --no-restore
dotnet run --project src/Verify.TUnit.Tests/Verify.TUnit.Tests.csproj -c Release --no-build --no-restore

# Fixie has no MTP runner — run via the fixie.console tool from the `src` directory
# (first: dotnet tool restore --tool-manifest src/.config/dotnet-tools.json)
cd src && dotnet fixie Verify.Fixie.Tests -c Release --no-build
```

### Running a single test

MTP does not accept the VSTest `--filter "FullyQualifiedName~..."` syntax. Build the project for a single framework, then run the compiled test executable directly with the xUnit v3 `-class` / `-method` filters (leading/trailing `*` wildcard supported):

```bash
dotnet build src/Verify.Tests/Verify.Tests.csproj -f net11.0 -c Debug
src/Verify.Tests/bin/Debug/net11.0/Verify.Tests.exe -class "PluginConventionTests"
src/Verify.Tests/bin/Debug/net11.0/Verify.Tests.exe -method "*StaleUnreferencedPluginIsSkipped*"
```

### Common Test Projects

- `Verify.Tests` - Core verification engine tests
- `Verify.Xunit.Tests` / `Verify.XunitV3.Tests` - xUnit adapter tests
- `Verify.NUnit.Tests` - NUnit adapter tests
- `Verify.MSTest.Tests` - MSTest adapter tests
- `Verify.Fixie.Tests` - Fixie adapter tests
- `Verify.Expecto.Tests` / `Verify.Expecto.FSharpTests` - Expecto (F#) adapter tests
- `Verify.TUnit.Tests` - TUnit adapter tests
- `DeterministicTests` - Deterministic serialization tests
- `StaticSettingsTests` - Global configuration tests
- `StrictJsonTests` - JSON parsing tests

### Tool Installation

```bash
# Restore dotnet tools (required for Fixie tests)
dotnet tool restore --tool-manifest src/.config/dotnet-tools.json
```

## Architecture

### Three-Layer Design

1. **Test Framework Adapters** (`Verify.NUnit`, `Verify.Xunit`, `Verify.MSTest`, etc.)
   - Extract test metadata (type name, method name, parameters) from framework context
   - Implement `BuildVerifier()` to create `InnerVerifier` instances
   - Each adapter is framework-specific but follows the same pattern

2. **Core Verification Engine** (`Verify` project)
   - `InnerVerifier` - Orchestrates verification workflow (split across 12 partial files by data type)
   - `VerifyEngine` - Comparison logic, diff tool launching, callback execution
   - `SettingsTask` - Fluent builder API for configuration

3. **Serialization & Comparison**
   - `VerifyJsonWriter` - Custom JSON writer extending Argon library
   - `SerializationSettings` - Registry of converters and scrubbers
   - `Comparer` / `FileComparer` / `StreamComparer` - Different comparison strategies

### Key Design Patterns

**Adapter Pattern**: Each test framework has a minimal adapter that extracts framework metadata differently:
- **Xunit**: Uses `UseVerifyAttribute` (a `BeforeAfterTestAttribute`) injected via MSBuild
- **NUnit**: Direct access to `TestContext.CurrentContext.Test`
- **MSTest**: Requires `[UsesVerify]` attribute with source generator
- **Fixie**: Uses `ExecutionState.Current`

**Partial Classes**: `InnerVerifier` is split across 12 files:
- `InnerVerifier_Object.cs` - Object serialization
- `InnerVerifier_String.cs` - Text comparison
- `InnerVerifier_Stream.cs` - Binary files
- `InnerVerifier_Json.cs` - JSON handling
- `InnerVerifier_Xml.cs` - XML serialization
- `InnerVerifier_Archive.cs` - ZIP files
- `InnerVerifier_Directory.cs` - Directory trees
- `InnerVerifier_File.cs` - Individual files
- `InnerVerifier_Task.cs` - Async unwrapping
- `InnerVerifier_Throws.cs` - Exception capture
- And others...


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [VerifyTests/Verify](https://github.com/VerifyTests/Verify) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
