---
trigger: always_on
description: Generates properties from `[VersionField]` fields (`m_Health` → `Health` property, prefix stripped + first letter capitalized):
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

ReactiveBinding is a C# Source Generator that provides compile-time reactive data binding for Unity. It generates change detection code at compile time, eliminating runtime reflection overhead.

### Usage Pattern

1. Class implements `IReactiveObserver`, marked `partial`
2. `[ReactiveSource]` marks data sources (fields, properties, methods)
3. `[ReactiveBind(nameof(Source))]` marks callbacks, or `[ReactiveBind]` for auto-inference
4. Call `ObserveChanges()` each frame (e.g., in `Update()`) to detect changes and trigger callbacks
5. Optional: `[ReactiveThrottle(N)]` controls check frequency, `[ReactiveObserveIgnore]` ignores call check, `ResetChanges()` for object pooling

### Key Features

- **Zero reflection** - All code generated at compile time
- **Polling model** - No event subscription/unsubscription, just call `ObserveChanges()`
- **Auto-inference** - `[ReactiveBind]` without parameters auto-detects referenced sources from method body
- **Flexible callbacks** - 0, N, or 2N parameters (old/new value pairs)
- **Inheritance** - Derived classes chain via `base.ObserveChanges()` automatically
- **VersionField** - `[VersionField]` auto-generates properties with version tracking and parent chain propagation
- **Custom property attributes** - `[VersionFieldProperty]` adds custom attributes to generated properties (Type for parameterless, string for parameterized)
- **Version containers** - `VersionList<T>`, `VersionDictionary<K,V>`, `VersionHashSet<T>` for efficient collection change detection
- **Data synchronization** - `[VersionSync]` on `[VersionField]` fields generates `IVersionSyncable` (`Commit`/`GetFull`/`GetDelta`/`Apply`/`ApplyDelta`) for full + field-level incremental `byte[]` sync
- **35 diagnostics** - Compile-time error/warning codes catch mistakes early

## Build Commands

```bash
dotnet build NuGet/ReactiveBinding.Generator/ReactiveBinding.Generator.csproj
dotnet test NuGet/ReactiveBinding.Tests/ReactiveBinding.Generator.Tests.csproj
dotnet test NuGet/ReactiveBinding.Tests/ReactiveBinding.Generator.Tests.csproj --filter "FullyQualifiedName~TestClassName.TestMethodName"
```

The generator DLL is automatically copied to `Unity/Runtime/Plugins/` after build via a PostBuild target.

## Repo layout

The repo splits into two parallel top-level directories:
- `Unity/` - the Unity (UPM) package (`package.json`, `Runtime/` + `.meta`, `Samples~/`). Install via `...ReactiveBinding.git?path=Unity`.
- `NuGet/` - the .NET side: `ReactiveBinding.Generator/` (generator), `ReactiveBinding.Tests/` (NUnit), `ReactiveBinding.Package/` (NuGet packaging), `.sln`, `Directory.Build.props`. Outside the Unity package, so Unity never compiles it.

The runtime C# source has a single shared copy under `Unity/Runtime/`; the NuGet projects reference it via `<Compile Include="..\..\Unity\Runtime\**\*.cs">`.

## NuGet packaging

`NuGet/ReactiveBinding.Package/ReactiveBinding.Package.csproj` packs the runtime (`Unity/Runtime/**/*.cs` → `lib/`) plus the generator (→ `analyzers/dotnet/cs/`) into a single NuGet package `XuToWei.ReactiveBinding`. Published to nuget.org on `v*` tags via `.github/workflows/publish-nuget.yml` using Trusted Publishing/OIDC (no stored key; needs a nuget.org Trusted Publishing policy + repo variable `NUGET_USER`). See `NuGet/ReactiveBinding.Package/README.md` for the full publishing guide. Manual: `dotnet pack NuGet/ReactiveBinding.Package/ReactiveBinding.Package.csproj -c Release`.

## Architecture

### Directory Structure
- `Unity/Runtime/` - Attributes, interfaces (`IReactiveObserver`, `IVersion`), version containers (`VersionList<T>`, `VersionDictionary<K,V>`, `VersionHashSet<T>`); `Unity/Runtime/Sync/` holds `IVersionSyncable` and `[VersionSync]`
- `NuGet/ReactiveBinding.Generator/` - Generator implementation
- `NuGet/ReactiveBinding.Tests/` - NUnit tests

### Core Components

**Generators** (`ISourceGenerator`):
- **ReactiveBindGenerator** - Generates `ObserveChanges()` and `ResetChanges()` from `[ReactiveSource]`/`[ReactiveBind]`
- **VersionFieldGenerator** - Generates properties from `[VersionField]` fields with `IVersion` implementation; also generates `IVersionSyncable` (`Commit`/`GetFull`/`GetDelta`/`Apply`/`ApplyDelta` + recursion primitives `WriteFull`/`WriteDelta`/`ReadFull`/`ReadDelta`/`ResetSync`) for classes with `[VersionSync]` fields

**Syntax Receivers** (`ISyntaxContextReceiver`):
- **ReactiveSyntaxReceiver** - Collects `[ReactiveSource]`/`[ReactiveBind]`/`[ReactiveThrottle]`, builds `ReactiveClassData`
- **VersionFieldSyntaxReceiver** - Collects `[VersionField]` fields, `[VersionFieldProperty]` and `[VersionSync]` markers

**Analyzers** (`DiagnosticAnalyzer`):
- **ReservedMethodAnalyzer** - Prevents manual `ObserveChanges()`/`ResetChanges()` (RB1005/RB1006)
- **ObserveChangesCallAnalyzer** - Warns when `ObserveChanges()` not called in class (RB0003), ignored by `[ReactiveObserveIgnore]` or reactive base class
- **ParentAccessAnalyzer** - Prevents `IVersion.Parent` access outside `IVersion` implementations (VF3001)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [XuToWei/ReactiveBinding](https://github.com/XuToWei/ReactiveBinding) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-20 -->
