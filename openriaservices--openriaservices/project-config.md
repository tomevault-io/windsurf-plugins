---
trigger: always_on
description: OpenRiaServices is a .NET Foundation project evolved from WCF RIA Services. It provides client/server libraries, hosting packages, code-generation tooling, Visual Studio integration, and tests for rich n-tier applications.
---

# Copilot instructions for OpenRiaServices

## Repository overview

OpenRiaServices is a .NET Foundation project evolved from WCF RIA Services. It provides client/server libraries, hosting packages, code-generation tooling, Visual Studio integration, and tests for rich n-tier applications.

The main solution is `D:\a\OpenRiaServices\OpenRiaServices\src\RiaServices.sln`.

Important areas:

- `src\OpenRiaServices.Client*` - client libraries and client-side tests.
- `src\OpenRiaServices.Server*` - server libraries, Entity Framework/Entity Framework Core integration, authentication, and tests.
- `src\OpenRiaServices.Hosting.*` - hosting implementations. Prefer ASP.NET Core for new work; WCF packages are legacy/deprecated.
- `src\OpenRiaServices.Tools*` - code-generation, MSBuild task, and T4/text-template tooling.
- `src\VisualStudio` - Visual Studio extension, item templates, project templates, and related tests.
- `src\Test` - shared test assets, integration/end-to-end tests, websites, and test databases.
- `NuGet` - package specifications and packaging scripts.
- `docs` - legacy documentation derived from WCF RIA Services documentation.

## Environment requirements

This repository is Windows-oriented. Use a Windows runner/shell for normal build and test work because the solution includes .NET Framework, Visual Studio, SQL LocalDB, VS extension/template, and Windows-targeted projects.

Expected tools:

- Visual Studio/MSBuild with .NET Framework 4.7.2 targeting support.
- .NET SDK 10.0.100 or newer. `src\global.json` requests `10.0.100` with `rollForward: major`.
- .NET 8 SDK/runtime for multi-targeted projects.
- NuGet/MSBuild restore support.
- SQL Server LocalDB (`(localdb)\MSSQLLocalDB`) for database-backed tests.
- PowerShell. `Setup-TestDatabases.ps1` may install/import the PowerShell `SqlServer` module unless run with `-UseSqlCmd`.

The cloud agent setup workflow already exists at `.github\workflows\copilot-setup-steps.yml` and selects `windows-latest`.

## Build commands

Prefer commands from the repository root, `D:\a\OpenRiaServices\OpenRiaServices`.

Restore and build the full solution:

```powershell
msbuild src\RiaServices.sln -restore /p:Configuration=Release /m /v:minimal
```

Alternative when MSBuild is correctly available through the .NET SDK and installed workloads:

```powershell
dotnet build src\RiaServices.sln --configuration Release
```

CI builds `src\RiaServices.sln` in `Release|Any CPU` on a Windows image with Visual Studio, .NET 8, .NET 10, NuGet, GitVersion, SQL LocalDB, and VSTest.

## Test commands

Most tests use MSTest. Some tests need SQL LocalDB databases restored from backups under `src\Test\Databases`.

Before running database-backed tests:

```powershell
sqllocaldb start MSSQLLocalDB
.\Setup-TestDatabases.ps1
```

If the `SqlServer` PowerShell module cannot be installed/imported but `sqlcmd` is available:

```powershell
.\Setup-TestDatabases.ps1 -UseSqlCmd
```

Run tests after building:

```powershell
dotnet test src\RiaServices.sln --configuration Release --settings src\test.runsettings
```

For focused changes, prefer running the relevant test project directly, for example:

```powershell
dotnet test src\OpenRiaServices.Hosting.AspNetCore\Test\OpenRiaServices.Hosting.AspNetCore.Test\OpenRiaServices.Hosting.AspNetCore.Test.csproj --configuration Release
```

CI separates test execution by target framework and excludes `OpenRiaServices.Common*Test.dll` from its main patterns. If a local full test run behaves differently from CI, inspect `azure-pipelines.yml` for the exact VSTest assembly patterns.

## Coding conventions

- Follow `.editorconfig` and existing style in nearby files.
- C# uses 4-space indentation and C# 14.0 (`src\Directory.Build.props`).
- Framework projects enable recommended analysis mode and XML documentation generation.
- New public, protected, and internal APIs should have XML documentation comments, this is not required for assembly local ("private") types.
- Release builds should not introduce warnings.
- Assembly signing is enabled centrally in `src\Directory.Build.props`; framework and test projects use keys from `src\snk`.
- Use existing libraries and patterns. Do not modernize unrelated legacy code while fixing a targeted issue.

## Versioning and packaging

- Version defaults and package metadata live in `src\Directory.Build.props`.
- CI uses GitVersion (`GitVersion.yml`) to derive build versions.
- NuGet package outputs go under `NuGet\bin` locally, or the artifact staging directory in CI.
- Packaging scripts and `.nuspec` files are under `NuGet`.

## Changelog and documentation expectations

- Treat changelog and documentation updates as part of completing a feature or notable behavior change, not as optional follow-up work.
- Record notable changes in the repository root changelog, `D:\a\OpenRiaServices\OpenRiaServices\Changelog.md`.
- Follow the existing changelog structure and keep entries compatible with the project's "Keep a Changelog" style release notes.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [OpenRIAServices/OpenRiaServices](https://github.com/OpenRIAServices/OpenRiaServices) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->
