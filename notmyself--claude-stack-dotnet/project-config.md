---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

ClaudeStack.net is a .NET 10.0 RC 2 template project with integrated Claude Code AI assistance, demonstrating full-stack architecture with centralized package management. The project includes ASP.NET Core MVC and Minimal API applications, each with corresponding unit tests (MSTest) and end-to-end tests (Playwright).

## Build & Test Commands

### Building
```bash
# Build entire solution
dotnet build

# Build specific project
dotnet build src/ClaudeStack.Web/ClaudeStack.Web.csproj
dotnet build src/ClaudeStack.API/ClaudeStack.API.csproj
```

### Running Applications
```bash
# Run MVC application
dotnet run --project src/ClaudeStack.Web/ClaudeStack.Web.csproj

# Run API application
dotnet run --project src/ClaudeStack.API/ClaudeStack.API.csproj
```

### Running Tests
```bash
# Run all tests
dotnet test

# Run specific test project (using Microsoft.Testing.Platform)
dotnet run --project tests/ClaudeStack.Web.Tests/ClaudeStack.Web.Tests.csproj
dotnet run --project tests/ClaudeStack.API.Tests/ClaudeStack.API.Tests.csproj

# Run Playwright tests
dotnet run --project tests/ClaudeStack.Web.Tests.Playwright/ClaudeStack.Web.Tests.Playwright.csproj
dotnet run --project tests/ClaudeStack.API.Tests.Playwright/ClaudeStack.API.Tests.Playwright.csproj

# Run a single test (using test filter)
dotnet test --filter FullyQualifiedName~TestMethod1
```

### Playwright Setup
After creating a new Playwright test project, install browsers:
```powershell
pwsh -Command "cd tests/ClaudeStack.Web.Tests.Playwright/bin/Debug/net10.0; ./playwright.ps1 install"
```

## Architecture

### Project Structure
- **src/ClaudeStack.Web**: ASP.NET Core MVC application with Razor runtime compilation enabled
- **src/ClaudeStack.API**: ASP.NET Core Web API using Minimal APIs with OpenAPI/Swagger
- **tests/ClaudeStack.Web.Tests**: MSTest unit tests for MVC application
- **tests/ClaudeStack.Web.Tests.Playwright**: Playwright end-to-end tests for MVC application
- **tests/ClaudeStack.API.Tests**: MSTest unit tests for API application
- **tests/ClaudeStack.API.Tests.Playwright**: Playwright end-to-end tests for API application

### Configuration Files

#### Directory.Build.props
Defines shared MSBuild properties for all projects:
- **TargetFramework**: net10.0
- **Nullable**: disabled (explicit using statements required)
- **ImplicitUsings**: disabled (explicit using statements required)
- **TreatWarningsAsErrors**: true

#### Directory.Packages.props
Centralized NuGet package version management (CPM). All package versions are defined here, and project files reference packages without versions.

To add a new package:
1. Add `<PackageVersion Include="PackageName" Version="x.y.z" />` to Directory.Packages.props
2. Add `<PackageReference Include="PackageName" />` (without Version) to the project file

#### global.json
Pins the .NET SDK version and configures the test runner:
```json
{
  "sdk": {
    "version": "10.0.100-rc.2.25502.107",
    "rollForward": "latestFeature"
  },
  "test": {
    "runner": "Microsoft.Testing.Platform"
  }
}
```

### Testing Framework

This project uses **MSTest with Microsoft.Testing.Platform** (the new test runner, not the legacy VSTest). Test projects require:
```xml
<PropertyGroup>
  <EnableMSTestRunner>true</EnableMSTestRunner>
  <OutputType>Exe</OutputType>
</PropertyGroup>
```

Tests are configured for method-level parallelization in MSTestSettings.cs:
```csharp
[assembly: Parallelize(Scope = ExecutionScope.MethodLevel)]
```

## Important Notes

### Creating New Test Projects

**CRITICAL**: When creating new MSTest projects, do NOT use the `--test-runner` flag. The test runner is already configured in global.json, and using the flag will overwrite the entire global.json file, removing the SDK version configuration.

```bash
# WRONG - will overwrite global.json
dotnet new mstest -o tests/NewProject --test-runner Microsoft.Testing.Platform

# CORRECT - test runner inherited from global.json
dotnet new mstest -o tests/NewProject
```

After creating the test project, manually add to the .csproj:
```xml
<PropertyGroup>
  <EnableMSTestRunner>true</EnableMSTestRunner>
  <OutputType>Exe</OutputType>
</PropertyGroup>
```

### Implicit Usings Disabled

Since ImplicitUsings is disabled, all C# files must include explicit using statements. Common namespaces needed:
- `using System;`
- `using System.Linq;`
- `using System.Threading.Tasks;`
- `using Microsoft.VisualStudio.TestTools.UnitTesting;` (for tests)
- `using Microsoft.Playwright.MSTest;` (for Playwright tests)
- ASP.NET Core namespaces (Microsoft.AspNetCore.Builder, Microsoft.Extensions.DependencyInjection, etc.)

### Package References

Always reference packages without Version attributes in .csproj files. Versions are centrally managed in Directory.Packages.props.

```xml
<!-- CORRECT -->
<PackageReference Include="Microsoft.AspNetCore.OpenApi" />

<!-- WRONG -->
<PackageReference Include="Microsoft.AspNetCore.OpenApi" Version="10.0.0" />
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NotMyself/claude-stack-dotnet](https://github.com/NotMyself/claude-stack-dotnet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
