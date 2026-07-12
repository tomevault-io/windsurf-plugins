---
trigger: always_on
description: These instructions define the conventions for ElBruno's .NET NuGet library repositories. Apply them when creating, modifying, or reviewing any project in this repo.
---

# ElBruno .NET Repository Conventions

These instructions define the conventions for ElBruno's .NET NuGet library repositories. Apply them when creating, modifying, or reviewing any project in this repo.

---

## Repository Structure

```
{ProjectName}/
├── README.md                    # Project overview, badges, quick start
├── LICENSE                      # MIT License
├── Directory.Build.props        # Shared MSBuild properties (all projects)
├── global.json                  # SDK version with rollForward
├── .gitignore                   # .NET + IDE + OS ignores
├── {ProjectName}.slnx           # XML-based solution file
├── src/
│   ├── {Owner}.{ProjectName}.{Package}/   # Library projects
│   ├── tests/{Package}.Tests/             # xUnit test projects
│   └── samples/{SampleName}/              # Sample/demo apps
├── docs/                        # All documentation except README.md and LICENSE
├── images/                      # All images including nuget_logo.png
└── .github/workflows/           # CI/CD workflows
```

- **Root files only:** README.md, LICENSE, Directory.Build.props, global.json, .gitignore, solution file
- **All source code** (libraries, tests, samples) lives under `src/`
- **All documentation** (except README.md and LICENSE) lives under `docs/`
- **All images** live under `images/`, including `nuget_logo.png` for NuGet packages

---

## Solution Format

- Use `.slnx` (XML-based solution format), **not** `.sln`
- All projects (libraries, tests, samples) must be included in the solution file
- Use `<Folder>` elements for logical grouping: `/src/`, `/src/tests/`, `/src/samples/`

---

## Project Conventions

### Library Projects
- Target `net8.0;net10.0` (multi-target: LTS + latest)
- Package naming: `{Owner}.{ProjectName}.{Feature}` (e.g., `ElBruno.QRCodeGenerator.Svg`)
- Each packable project must include the NuGet icon:
  ```xml
  <None Include="$(MSBuildThisFileDirectory)..\..\images\nuget_logo.png" Pack="true" PackagePath="" />
  ```
- Include `<InternalsVisibleTo>` for the corresponding test project
- Enable symbol packages: `<IncludeSymbols>true</IncludeSymbols>`, `<SymbolPackageFormat>snupkg</SymbolPackageFormat>`
- Enable deterministic CI builds: `<ContinuousIntegrationBuild Condition="'$(GITHUB_ACTIONS)' == 'true'">true</ContinuousIntegrationBuild>`

### Test Projects
- Target `net8.0` only (single target)
- Framework: xUnit with coverlet.collector
- Naming: `{LibraryProject}.Tests`
- Location: `src/tests/{LibraryProject}.Tests/`
- Set `<IsPackable>false</IsPackable>` and `<IsTestProject>true</IsTestProject>`

### Tool Projects (dotnet global tools)
- Target `net8.0` only (single target — CI runners may not have preview SDKs)
- Set `<PackAsTool>true</PackAsTool>` and `<ToolCommandName>{toolname}</ToolCommandName>`

### Sample Projects
- Location: `src/samples/{SampleName}/`
- Reference library projects via `<ProjectReference>`

---

## Directory.Build.props

Place at repo root. Shared by all projects:

```xml
<Project>
  <PropertyGroup>
    <!-- Language -->
    <LangVersion>latest</LangVersion>
    <Nullable>enable</Nullable>
    <ImplicitUsings>enable</ImplicitUsings>
    <TreatWarningsAsErrors>false</TreatWarningsAsErrors>
    <WarningsAsErrors>nullable</WarningsAsErrors>

    <!-- Code analysis -->
    <EnforceCodeStyleInBuild>true</EnforceCodeStyleInBuild>
    <EnableNETAnalyzers>true</EnableNETAnalyzers>
    <AnalysisLevel>latest</AnalysisLevel>

    <!-- Repository information -->
    <RepositoryUrl>https://github.com/{owner}/{repo}</RepositoryUrl>
    <RepositoryType>git</RepositoryType>
    <PublishRepositoryUrl>true</PublishRepositoryUrl>

    <!-- Package defaults -->
    <Authors>Bruno Capuano (ElBruno)</Authors>
    <Company>Bruno Capuano</Company>
    <Copyright>Copyright © Bruno Capuano $([System.DateTime]::Now.Year)</Copyright>
    <PackageLicenseExpression>MIT</PackageLicenseExpression>
    <PackageIcon>nuget_logo.png</PackageIcon>
  </PropertyGroup>
</Project>
```

---

## global.json

Use `"rollForward": "latestMajor"` for flexibility across dev machines and CI:

```json
{
  "sdk": {
    "version": "8.0.0",
    "rollForward": "latestMajor"
  }
}
```

---

## .gitignore

Standard .NET patterns:

```gitignore
# Build outputs
[Bb]in/
[Oo]bj/
[Dd]ebug/
[Rr]elease/

# NuGet
*.nupkg
*.snupkg
.nuget/
packages/

# IDE
.vs/
*.user
*.suo
*.cache
*.DotSettings.user

# OS
Thumbs.db
.DS_Store
```

---

## GitHub Actions — CI Build (`build.yml`)

- **Triggers:** push to `main`, PR to `main`
- **Runner:** `ubuntu-latest`
- **SDK:** `dotnet-version: 8.0.x`
- **Commands use solution-level operations** (not per-project)
- **Use `-p:TargetFrameworks=net8.0`** for restore and build (CI runners may not have preview SDKs)
- **Use `--framework net8.0`** for test

```yaml
name: CI Build

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Setup .NET
        uses: actions/setup-dotnet@v4
        with:
          dotnet-version: 8.0.x
      - name: Restore
        run: dotnet restore {ProjectName}.slnx -p:TargetFrameworks=net8.0
      - name: Build

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [elbruno/ElBruno.Whisper](https://github.com/elbruno/ElBruno.Whisper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-12 -->
