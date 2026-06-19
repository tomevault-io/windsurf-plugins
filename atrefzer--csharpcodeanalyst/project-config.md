---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Platform & Runtime

- Windows-only WPF application (`CSharpCodeAnalyst.csproj` targets `net10.0-windows`, `OutputType=WinExe`). Building / running on Linux/macOS is not supported.
- Requires **.NET 10 SDK** (build) and the **.NET 10 Runtime** plus **MSBuild** from a Visual Studio or .NET SDK install (the parser loads solutions through `MSBuildWorkspace`).
- Line endings: the repo is full of Windows-native WPF assets; do not reformat or convert CRLFs.

## Common commands

Run from the repository root.

```bash
# Restore and build the whole solution
dotnet restore
dotnet build

# Release build with explicit version (matches CI)
dotnet build --no-restore -c Release -p:Version=0.9.0 -p:FileVersion=0.9.0 -p:AssemblyVersion=0.9.0

# Run all tests
dotnet test --configuration Release

# Run a single NUnit fixture or test (filter by fully-qualified name)
dotnet test Tests/Tests.csproj --filter "FullyQualifiedName~CyclesApprovalTests"
dotnet test Tests/Tests.csproj --filter "FullyQualifiedName=CodeParserTests.ApprovalTests.CyclesApprovalTests.MethodName"

# Publish the WPF app (framework-dependent, win-x64, same flags as the release pipeline)
dotnet publish .\CSharpCodeAnalyst\CSharpCodeAnalyst.csproj -c Release -r win-x64 -o publish `
  --self-contained false -p:PublishSingleFile=false -p:SatelliteResourceLanguages=en
```

Command-line (headless) validation mode — triggered when more than one CLI arg is passed, so `App.OnStartup` skips the UI and exits with a status code:

```
CSharpCodeAnalyst.exe -validate -sln:<path.sln> -rules:<path.txt> [-log-console] [-log-file:<file>] [-out:<file>]
```

Exit codes: `0` = clean, `1` = violations, `2` = validation failed. See `Documentation/command-line-arguments.md`.

Debug shortcut: passing a single arg `-load:<project.json>` auto-loads a saved project after the UI starts (see `App.LoadProjectFileFromCommandLineAsync`).

## Solution layout

Five projects wired together in `CSharpCodeAnalyst.sln`:

- **`CodeGraph/`** — pure, UI-free domain model. Contains the `CodeElement` / `Relationship` / `CodeGraph` graph types (`Graph/`), graph algorithms (`Algorithms/Cycles`, `Algorithms/Metrics`, `Algorithms/Partitioning`), exporters (`Export/` — DGML, DSI, PlantUML, JSON), and `Exploration/CodeGraphExplorer` (traversal queries used from the UI context menus). No WPF dependencies — reference this project from tests and tools.
- **`CodeParser/`** — Roslyn front-end that turns an `.sln` or `.csproj` into a `CodeGraph`. Entry point: `Parser.ParseAsync(path)`. Works in **two passes**: `HierarchyAnalyzer` finds code elements and parent/child links, then `RelationshipAnalyzer.AnalyzeRelationships` walks method and lambda bodies to build relationships (parallel by default; pass `maxDegreeOfParallelism: 1` for a single-threaded debug run). `Initializer.InitializeMsBuildLocator()` **must** be called once before any parse (both `App.StartUi` and the test fixture `Init` do this).
- **`CSharpCodeAnalyst/`** — WPF front-end. Organized by feature under `Features/` (`CycleGroups`, `Graph`, `Tree`, `AdvancedSearch`, `Analyzers/ArchitecturalRules`, `Analyzers/EventRegistration`, `Ai`, `Import`, `Export`, `Metrics`, `Partitions`, `Refactoring`, `Gallery`, `Help`, `Info`). Cross-cutting infrastructure lives in `Shared/` (messaging, notifications, data grid, search, filter, WPF helpers). `Configuration/` holds `AppSettings` (from `appsettings.json`), `UserPreferences` (persisted to `userSettings.json`), and `AiCredentialStorage`. Persistence of saved projects is in `Persistence/` (JSON, with DTOs under `Dto/`).
- **`Tests/`** (project name `CodeParserTests`) — NUnit suite. `ApprovalTests/` parses the `TestSuite/` C# solution once per fixture and asserts on the resulting graph; `UnitTests/` covers cycles, exploration, export, search, architectural rules, etc.
- **`ApprovalTestTool/`** — standalone console app that clones external repos listed in `Repositories.txt`, parses each at a pinned commit, hashes the graph dump, and diffs against references. Used to catch parser regressions on real codebases; not part of the CI test run.

`TestSuite/` is a handcrafted C# solution used purely as parser input for the approval tests. Do not consume it from production code — it is intentionally full of odd language constructs. `ReferencedAssemblies/` contains the MSAGL DLLs referenced directly by `CSharpCodeAnalyst.csproj` and `Tests.csproj` (MSAGL is not on NuGet for the versions used here).

## Architectural notes worth knowing before editing

### MSBuild runtime trap (`Directory.Build.props`)
Every project inherits a `Microsoft.Build.Framework` `PackageReference` with `ExcludeAssets="runtime"`. This is load-bearing: `Microsoft.Build.Locator` loads MSBuild from the installed SDK at runtime, and copying the NuGet-provided MSBuild DLLs into `bin/` causes `RPC_E_CALL_REJECTED` and other loader failures. When upgrading Roslyn / MSBuild packages, keep the exclusion in place and bump the version comment in `Directory.Build.props` to match the transitive version from `Microsoft.Build.Locator`.

### Parser: two passes, then global-namespace fixup

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ATrefzer/CSharpCodeAnalyst](https://github.com/ATrefzer/CSharpCodeAnalyst) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-19 -->
