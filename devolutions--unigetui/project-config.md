---
trigger: always_on
description: UniGetUI is an Avalonia desktop app (C#/.NET 10) providing a GUI for CLI package managers (WinGet, Scoop, Chocolatey, Pip, Npm, .NET Tool, PowerShell Gallery, Cargo, Vcpkg).
---

# UniGetUI - Copilot Instructions

## Project Overview

UniGetUI is an Avalonia desktop app (C#/.NET 10) providing a GUI for CLI package managers (WinGet, Scoop, Chocolatey, Pip, Npm, .NET Tool, PowerShell Gallery, Cargo, Vcpkg).

Solution entry points:
- `src/UniGetUI.Windows.slnx` - official Windows solution; builds the Avalonia app and Windows-specific package-manager integrations
- `src/UniGetUI.Avalonia.slnx` - cross-platform Avalonia solution

## Architecture

The codebase follows a **layered, modular structure** with ~40 projects:

- **`UniGetUI.Avalonia/`** - Avalonia entry point, AXAML pages, controls, and app shell (`Program.cs`, `Views/MainWindow.axaml`)
- **`SharedAssets/`** - shared app icons, symbols, splash images, and utility scripts used by packaging
- **`UniGetUI.Core.*`** - Shared infrastructure: `Logger`, `Settings`, `Tools` (includes `CoreTools.Translate()`), `IconEngine`, `LanguageEngine`
- **`UniGetUI.PackageEngine.Interfaces`** - Contracts: `IPackageManager`, `IPackage`, `IManagerSource`, `IPackageDetails`
- **`UniGetUI.PackageEngine.PackageManagerClasses`** - Base implementations: `PackageManager` (abstract), `Package`, helpers (`BasePkgDetailsHelper`, `BasePkgOperationHelper`, `BaseSourceHelper`)
- **`UniGetUI.PackageEngine.Managers.*`** - Concrete manager implementations (one project per manager: `WinGet`, `Scoop`, `Chocolatey`, `Pip`, `npm`, etc.)
- **`UniGetUI.PackageEngine.Operations`** - Install/update/uninstall operation orchestration
- **`UniGetUI.Interface.*`** - Enums, telemetry, background API

## Adding a New Package Manager

Each manager extends `PackageManager` and must override three abstract methods:

```csharp
protected override IReadOnlyList<Package> FindPackages_UnSafe(string query);
protected override IReadOnlyList<Package> GetAvailableUpdates_UnSafe();
protected override IReadOnlyList<Package> GetInstalledPackages_UnSafe();
```

Each manager also provides three helper classes (in a `Helpers/` subfolder):
- `*PkgDetailsHelper` extends `BasePkgDetailsHelper` - overrides `GetDetails_UnSafe`, `GetInstallableVersions_UnSafe`, `GetIcon_UnSafe`, etc.
- `*PkgOperationHelper` extends `BasePkgOperationHelper` - overrides `_getOperationParameters`, `_getOperationResult`
- `*SourceHelper` extends `BaseSourceHelper` - overrides `GetSources_UnSafe`, `GetAddSourceParameters`, etc.

The constructor sets `Capabilities`, `Properties`, and wires the helpers. See `src/UniGetUI.PackageEngine.Managers.Scoop/Scoop.cs` as a clean reference implementation.

## Build & Test

```shell
# Restore & test (from src/)
dotnet restore UniGetUI.Windows.slnx
dotnet test UniGetUI.Windows.slnx --verbosity q --nologo /p:Platform=x64

# Publish release build
dotnet publish src/UniGetUI.Avalonia/UniGetUI.Avalonia.csproj /p:Configuration=Release /p:Platform=x64 -p:RuntimeIdentifier=win-x64
```

- Target framework: `net10.0-windows10.0.26100.0` (min `10.0.19041`)
- Build generates secrets via `src/UniGetUI.Avalonia/Infrastructure/generate-secrets.ps1` and integrity tree via `scripts/generate-integrity-tree.ps1`
- Self-contained, publish-trimmed (partial), Windows App SDK self-contained
- Tests use **xUnit** (`[Fact]`, `Assert.*`)

## NativeAOT and Trim Safety

Release packages are self-contained, fully trimmed NativeAOT binaries on every supported RID. Treat NativeAOT safety as a non-negotiable production requirement: changes must work without runtime-generated code or metadata that the trimmer cannot prove is required.

- Consider every `IL2xxx` trim warning and `IL3xxx` AOT warning a defect. Do not blanket-suppress warnings or use a broad `RequiresUnreferencedCode`, `RequiresDynamicCode`, `DynamicDependency`, `DynamicallyAccessedMembers`, linker descriptor, or root assembly as a workaround. An exception must be narrowly scoped, explain the concrete runtime requirement, and have a publish-path test that proves it is safe.
- Use `JsonSerializerContext`/`JsonTypeInfo<T>` source-generated metadata for every production JSON serialization or deserialization path, including generic helpers, HTTP payloads, settings, IPC, telemetry, and persisted state. Do not call reflection-based serializer overloads or retain a reflection fallback for unknown application types. Add the closed type to a context and test the persisted or transported path.
- Do not introduce production reflection-driven activation or discovery (`Assembly.Load`, `Type.GetType`, `Activator`, member lookup/invocation), runtime generic construction, expression compilation, runtime code generation, or dynamically loaded plugins. Prefer explicit registrations, direct constructors, generated metadata, and compiled XAML bindings. If a framework-owned dynamic boundary cannot be removed, isolate it and add published-NativeAOT runtime coverage.
- New shipping COM interop must use source-generated interfaces and wrappers (`GeneratedComInterface`, `StrategyBasedComWrappers`) or raw ABI calls; do not add `ComImport`, `CoClass`, RCW activation, or legacy marshal-to-object APIs to a NativeAOT path. Define the ABI precisely, balance COM reference ownership, and test both x64 and arm64 where applicable.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Devolutions/UniGetUI](https://github.com/Devolutions/UniGetUI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
