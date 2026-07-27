---
trigger: always_on
description: The Microsoft Performance Toolkit SDK is a cross-platform C# library that enables developers to create "SDK plugins" for processing and analyzing performance data from arbitrary data sources (ETL files, CTF files, SQL logs, CSV files, etc.). These plugins can be used by performance analysis applications like Windows Performance Analyzer (WPA).
---

# Microsoft Performance Toolkit SDK - Copilot Instructions

## Repository Overview
The Microsoft Performance Toolkit SDK is a cross-platform C# library that enables developers to create "SDK plugins" for processing and analyzing performance data from arbitrary data sources (ETL files, CTF files, SQL logs, CSV files, etc.). These plugins can be used by performance analysis applications like Windows Performance Analyzer (WPA).

**Repository Stats:**
- **Languages:** C# (.NET)
- **Project Type:** Class library SDK with NuGet packages
- **Target Frameworks:** .NET Standard 2.0, .NET Standard 2.1, .NET 6.0, .NET 8.0
- **Solution:** Single main solution at `src/SDK.sln` with 17+ projects
- **Size:** Medium-sized repository with SDK core, runtime, engine, plugins system, tools, and comprehensive test coverage

## Build and Test Instructions

### Prerequisites
- **.NET SDK 8.x** (or compatible version that supports .NET Standard 2.0)
- **Nerdbank.GitVersioning** tool version 3.6.143 (automatically restored via NuGet)
- **Windows OS** (primary target platform, uses Windows-specific APIs in some areas)

### Build Commands (VALIDATED - ALWAYS USE THESE)

**CRITICAL: Always run commands in the following order and configuration:**

1. **Restore Dependencies** (Optional but recommended for clean builds):
   ```powershell
   dotnet restore "src/SDK.sln"
   ```
   - Can be skipped; `dotnet build` will restore automatically
   - Always restore before building if you want `--no-restore` flag benefits

2. **Build the Solution** (REQUIRED):
   ```powershell
   dotnet build "src/SDK.sln" --no-restore -c Release
   ```
   - **Configuration:** Always use `-c Release` for production builds
   - **Expected Warnings:** 
     - ~22 warnings about obsolete interfaces (IApplicationEnvironmentV2, IApplicationEnvironmentV3) - THESE ARE EXPECTED
   - Build **will succeed** even with these warnings
   - Without `--no-restore`: Auto-restores first

3. **Run Tests** (REQUIRED for validation):
   ```powershell
   dotnet test "src/SDK.sln" -c Release --filter "TestCategory~Unit|TestCategory~Integration|TestCategory~Functional" --no-build --verbosity normal
   ```
   - **Expected Results:** 712+ tests pass, 0-1 skipped, 0 failures
   - **Filter:** Only runs Unit, Integration, and Functional tests (not all tests are categorized)
   - **Important:** Use `--no-build` to run against already-built assemblies
   - Some test projects show warnings about no matching tests - THIS IS EXPECTED for PluginsSystem test projects

4. **Build Sample Projects** (Optional validation):
   ```powershell
   Get-ChildItem "samples" -Filter *.sln -Recurse | ForEach-Object { dotnet build $_.FullName -c Release }
   ```
   - Builds `SimpleDataSource/SampleAddIn.sln` and `SqlPlugin/SqlPlugin.sln`
   - Validates plugin development patterns

### Clean Build
To start fresh:
```powershell
dotnet clean "src/SDK.sln"
dotnet build "src/SDK.sln" -c Release
```
Note: `dotnet build` without restore will auto-restore.

## Project Structure and Key Files

### Root Directory Structure
```
.github/              # CI/CD workflows and GitHub configuration
  workflows/
    ci.yml            # PR validation: build + test + samples
    main_build_status.yml   # Main branch build status
    main_tests_status.yml   # Main branch test status
src/                  # Main solution and all projects
  SDK.sln             # Primary solution file
  Directory.Build.props     # Shared MSBuild properties (Nerdbank.GitVersioning)
  Directory.Build.targets   # Shared MSBuild targets (test package versions)
  version.json        # Version configuration (Nerdbank.GitVersioning)
  nuget.config        # NuGet package source configuration
samples/              # Example SDK plugins
  SimpleDataSource/   # Basic text file processing plugin
  SqlPlugin/          # SQL trace file processing plugin
documentation/        # Comprehensive SDK usage documentation
  Using-the-SDK/      # Plugin development guides
  Architecture/       # SDK architecture deep-dives
devel-template/       # .NET template for creating SDK plugins (WIP)
```

### Key Projects in src/SDK.sln

**Core SDK Libraries (Plugins depend on these):**
- `Microsoft.Performance.SDK` - Core SDK library (.NET Standard 2.0) - **This is what plugins reference**
- `Microsoft.Performance.SDK.Runtime` - Runtime for loading/processing plugins (.NET Standard 2.0) - **Plugins should NOT depend on this**
- `Microsoft.Performance.SDK.Runtime.NetCoreApp` - .NET Core specific runtime (multi-targeted: net6.0, net8.0)
- `Microsoft.Performance.Toolkit.Engine` - Programmatic interface for manipulating plugin data (.NET Standard 2.1)

**Plugins System:**
- `Microsoft.Performance.Toolkit.Plugins.Core` - Core plugin system abstractions (.NET Standard 2.0)
- `Microsoft.Performance.Toolkit.Plugins.Runtime` - Plugin system runtime (.NET Standard 2.0)
- `Microsoft.Performance.Toolkit.Plugins.Cli` - CLI tool for plugin management (net8.0)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [microsoft/microsoft-performance-toolkit-sdk](https://github.com/microsoft/microsoft-performance-toolkit-sdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
