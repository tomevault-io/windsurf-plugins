---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Polyfill is a **source-only** NuGet package that exposes newer .NET and C# features to older runtimes. It ships as C# source files (not compiled assemblies) that get compiled directly into consuming projects. There are 723+ polyfilled APIs. A compiled library variant exists as `PolyfillLib`.

Targets `netstandard2.0` and supports: `net461`–`net481`, `netcoreapp2.0`–`3.1`, `net5.0`–`net11.0`, `uap10`.

## Build Commands

```bash
# Build the full solution
dotnet build src/Polyfill.slnx --configuration Release

# Build just the core projects (faster iteration)
dotnet build src/Polyfill.slnf --configuration Release

# Run all tests (requires build first)
pwsh ./src/run-tests.ps1

# Run a single test project for a specific framework
dotnet run --project src/Tests/Tests.csproj --configuration Release --framework net10.0 --no-build

# Run a single test by name (TUnit uses Microsoft.Testing.Platform)
# Use --treenode-filter (not --filter). * matches within a segment, ** only at the end.
# Pattern: /assembly/namespace/ClassName/MethodName
dotnet run --project src/Tests/Tests.csproj --framework net10.0 -- --treenode-filter "/*/*/PolyfillTests/StreamReaderReadAsync"

# Run all tests matching a prefix within a class
dotnet run --project src/Tests/Tests.csproj --framework net10.0 -- --treenode-filter "/*/*/PolyfillTests/Directory_*"
```

Tests use **TUnit** (not xUnit/NUnit). Test assertions use `await Assert.That(...)` syntax.

## Architecture

### Source-only Package Mechanism

The NuGet package ships `.cs` files (not a DLL). The `Polyfill.nuspec` packs files from `src/Split/` into `contentFiles/cs/`. The `Polyfill.targets` file is included in the package to set up conditional compilation constants at build time based on the consumer's target framework and referenced packages.

### Key Directories

- **`src/Polyfill/`** — The canonical source. Contains all polyfill implementations. This is the "master" copy.
- **`src/Split/{tfm}/`** — Per-target-framework copies of source files that ship in the NuGet. These are generated/maintained by `ApiBuilderTests` — do not edit directly.
- **`src/Tests/`** — Primary TUnit test project, multi-targets many frameworks.
- **`src/Consume/`** — Compilation-only project that ensures all APIs compile on all supported frameworks (no test assertions).
- **`src/Consume*/`** — All `Consume*` projects get their polyfill source files via `<Import Project="$(SolutionDir)\TestIncludes.targets" />`, which includes `Split/{tfm}/**/*.cs` based on the target framework. They have no polyfill `.cs` files of their own — the Split files ARE their compiled source.
- **`src/ApiBuilderTests/`** — Tests that generate the Split output and verify API surface.

### Conditional Compilation

Polyfill uses extensive `#if` directives. Key constants:

- **Framework constants**: `NETFRAMEWORK`, `NETSTANDARD`, `NETCOREAPP2X`, `NETCOREAPP3X`, `NET46X`, `NET47X`, `NET48X`
- **Feature constants** (set by `Polyfill.targets` based on referenced packages): `FeatureMemory`, `FeatureValueTask`, `FeatureValueTuple`, `FeatureRuntimeInformation`, `FeatureHttp`, `FeatureCompression`, `FeatureAsyncInterfaces`
- **Consumer options**: `PolyPublic` (makes APIs public), `PolyUseEmbeddedAttribute`, `PolyEnsure`, `PolyGuard`, `PolyNullability`, `PolyArgumentExceptions`, `PolyStringInterpolation`
- **`AllowUnsafeBlocks`** — enables unsafe polyfill variants for better performance

### Code Organization Pattern

- **Attributes** (e.g., `ModuleInitializerAttribute.cs`): standalone files, one per attribute
- **Extension methods** on existing types: `Polyfill_{TypeName}.cs` (e.g., `Polyfill_StreamWriter.cs`)
- **Static helper polyfills** (add a static method to an existing type via `extension(Type)`): `{TypeName}Polyfill.cs` (e.g., `EnumPolyfill.cs`)
- **Recreated BCL types** (a type that doesn't exist at all on older TFMs, e.g. `Base64Url`, `KeyValuePair`, `Lock`, `Index`): standalone file `{TypeName}.cs` declaring the type in its **real namespace** (not the `Polyfill` partial), wrapped `#if !NETx_0_OR_GREATER` ... `#else [assembly: TypeForwardedTo(typeof(...))] #endif`. Apply `[ExcludeFromCodeCoverage]`, `[DebuggerNonUserCode]`, and the `PolyUseEmbeddedAttribute`/`PolyPublic` gates.
- **Optional feature groups**: `Ensure/`, `Guard/`, `Nullability/`, `ArgumentExceptions/`, `StringInterpolation/`

**Important constraints:**
- Each `*Polyfill.cs` file must contain exactly **one top-level type** (the `Polyfill` partial class). Helper classes must be **nested** inside `Polyfill`, otherwise `ReadMethodsForFiles` in `BuildApiTest` will throw.
- The filename of static polyfill files directly determines the `api_list.include.md` section header: `{TypeName}Polyfill.cs` → `#### {TypeName}`. For example, `FilePolyfill.cs` → `#### File`. Choose filenames to match the type being extended.
- `//Link:` comments on public methods must use `?view=net-11.0` for learn.microsoft.com URLs (enforced by `LinkReader`). For overloaded methods, include the `#fragment` anchor pointing to the specific overload (e.g., `#system-type-method(system-string-system-int32)`).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SimonCropp/Polyfill](https://github.com/SimonCropp/Polyfill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
