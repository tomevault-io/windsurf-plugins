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

## ⚠️ Critical Rule: All Code Lives Under `src/`

**Every** project — libraries, tests, samples, benchmarks, tools — MUST be placed under the `src/` directory. Nothing gets created at the repo root except:
- README.md
- LICENSE
- Directory.Build.props
- global.json
- .gitignore
- .editorconfig
- {ProjectName}.slnx

**Structure under `src/`:**
- `src/{Owner}.{ProjectName}.{Feature}/` — Library projects (packable)
- `src/tests/{LibraryProject}.Tests/` — Test projects
- `src/samples/{SampleName}/` — Sample/demo apps
- `src/tools/{ToolName}/` — Tool projects (dotnet global tools)

**Golden rule: NEVER create project directories at the repo root. NEVER put tests or samples outside `src/`.**

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


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [elbruno/ElBruno.MarkItDotNet](https://github.com/elbruno/ElBruno.MarkItDotNet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
