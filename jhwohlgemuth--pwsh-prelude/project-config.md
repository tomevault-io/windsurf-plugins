---
trigger: always_on
description: Validates that the environment has necessary tools and dependencies for Prelude development.
---

# AGENT.md - Project Configuration for AI Agents

## Project Overview
**Prelude** is a PowerShell "standard" library providing utilities, helpers, functions, type accelerators, and aliases inspired by functional programming language preludes. It supports linear algebra, graph theory, data analysis, and system automation.

## Project Structure

### Main Directories
- **`Prelude/`** - PowerShell module source
  - `Prelude.psd1` - Module manifest
  - `Prelude.psm1` - Module entry point
  - `src/` - Source files organized by functionality
    - `core.ps1` - Core functions
    - `application.ps1` - Application utilities
    - `data.ps1` - Data manipulation
    - `graph.ps1` - Graph theory implementations
    - `matrix.ps1` - Linear algebra/matrix operations
    - `productivity.ps1` - Productivity helpers
    - `web.ps1` - Web utilities
  - `Plus/` - Extended functionality
  - `types/` - Custom type definitions
  - `formats/` - PowerShell formatting files
- **`csharp/`** - C# implementations for performance
  - `Tests/` - .NET unit tests
  - `Graph/`, `Matrix/`, etc. - Domain-specific implementations
- **`tests/`** - PowerShell Pester tests
  - `*.Tests.ps1` - Test files corresponding to `src/` modules
- **`docs/`**, **`examples/`** - Documentation and examples
- **`styles/`** - Code quality/linting rules (alex, proselint, etc.)

### Configuration Files
- `Prelude.psd1` - PowerShell module manifest (defines exports)
- `PSScriptAnalyzerSettings.psd1` - PSScriptAnalyzer configuration
- `stryker-config.json` - Mutation testing configuration
- `appveyor.yml` - CI/CD configuration
- `codecov.yml` - Code coverage configuration

## Development Conventions

### PowerShell Functions
- Located in `Prelude/src/` organized by feature area
- Functions follow verb-noun naming: `Get-Something`, `Set-Something`, `New-Something`
- Must be exported in `Prelude.psd1` under `FunctionsToExport`
- Should have Comment-Based Help (CBH) with `.SYNOPSIS`, `.DESCRIPTION`, `.PARAMETER`, `.EXAMPLE`

### C# Code
- Located in `csharp/` with corresponding project files (`.csproj`)
- Performance-critical code is implemented in C#
- Tests in `csharp/Tests/Tests.csproj` using standard .NET testing
- Classes follow PascalCase naming

### Testing
- PowerShell tests: `tests/*.Tests.ps1` using Pester framework
- .NET tests: `csharp/Tests/*.Tests.cs`
- Test files should mirror source file names
- Run tests with build tasks (see below)

### Code Quality
- PowerShell scripts analyzed with PSScriptAnalyzer
- Configuration: `PSScriptAnalyzerSettings.psd1`
- Custom rules: `PSScriptAnalyzerCustomRules.psm1`
- StyleCop used for C# (config: `stylecop.json`)

## Build & Test Tasks

Available VS Code tasks (run with `Ctrl+Shift+B` or via task runner):
- **build** - Compile .NET projects: `dotnet build csharp/Tests/Tests.csproj`
- **watch** - Watch mode: `dotnet watch run csharp/Tests/Tests.csproj`
- **publish** - Publish: `dotnet publish csharp/Tests/Tests.csproj`

### Using Invoke-Task for Testing and Linting

The main task execution script is `./Invoke-Task.ps1`. Run any task using switch parameters:

**Setup Scripts:**
- `./Invoke-Setup.ps1` - Install/configure development dependencies
- `./Invoke-FixPesterSetup.ps1` - Fix Pester installation issues

#### Available Tasks

**Lint - Analyze and format code**
```powershell
./Invoke-Task.ps1 -Lint                    # Lint PowerShell and C# code with auto-fix
./Invoke-Task.ps1 -Lint -Skip dotnet       # Lint only PowerShell code
./Invoke-Task.ps1 -Lint -Skip powershell   # Lint only C# code
./Invoke-Task.ps1 -Lint -DryRun            # Analyze without making changes
./Invoke-Task.ps1 -Lint -CI                # Run in CI mode
```
Runs PSScriptAnalyzer on PowerShell (`Prelude/src/` and `Prelude/Plus/`) and dotnet-format on C# code. Configuration: `PSScriptAnalyzerSettings.psd1`, `PSScriptAnalyzerCustomRules.psm1`.

**Test - Run unit tests**
```powershell
./Invoke-Task.ps1 -Test                              # Run all tests
./Invoke-Task.ps1 -Test -Skip powershell             # Run only C# tests
./Invoke-Task.ps1 -Test -Skip dotnet                 # Run only PowerShell Pester tests
./Invoke-Task.ps1 -Test -WithCoverage                # Run tests with code coverage
./Invoke-Task.ps1 -Test -WithCoverage -GenerateCoverageReport  # Generate coverage report
./Invoke-Task.ps1 -Test -Tags Remote                 # Run only tests tagged 'Remote'
./Invoke-Task.ps1 -Test -Exclude WindowsOnly         # Exclude tests tagged 'WindowsOnly'
./Invoke-Task.ps1 -Test -Filter '*Readability*'      # Run tests matching filter pattern
./Invoke-Task.ps1 -Test -Platform linux              # Run tests for Linux
```
Runs Pester tests (`tests/*.Tests.ps1`) and .NET tests (`csharp/Tests/Tests.csproj`). Coverage reports can be viewed at `.\coverage\index.htm`.

**Build - Compile C# and create link libraries**
```powershell
./Invoke-Task.ps1 -Build                   # Format, test, and build link libraries
./Invoke-Task.ps1 -Build -BuildOnly        # Skip formatting and testing
./Invoke-Task.ps1 -Build -Version 2019     # Use Visual Studio 2019
./Invoke-Task.ps1 -Build -Architecture x86 # Build for 32-bit architecture
```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jhwohlgemuth/pwsh-prelude](https://github.com/jhwohlgemuth/pwsh-prelude) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
