---
trigger: always_on
description: These instructions define the conventions for the graphify-dotnet project. Apply them when creating, modifying, or reviewing any project in this repo.
---

# graphify-dotnet Repository Conventions

These instructions define the conventions for the graphify-dotnet project. Apply them when creating, modifying, or reviewing any project in this repo.

> **graphify-dotnet** is a .NET 10 console application published to NuGet as a [dotnet tool](https://www.nuget.org/packages/graphify-dotnet). The CLI is packaged via `PackAsTool` in `Graphify.Cli.csproj` and published through the `publish.yml` workflow.

---

## Repository Structure

```
graphify-dotnet/
├── README.md                    # Project overview, badges, quick start
├── LICENSE                      # MIT License
├── Directory.Build.props        # Shared MSBuild properties (all projects)
├── global.json                  # SDK version with rollForward
├── .gitignore                   # .NET + IDE + OS ignores
├── graphify-dotnet.slnx         # XML-based solution file
├── src/
│   ├── Graphify/                # Core library (graph pipeline, data structures, export)
│   ├── Graphify.Cli/            # CLI tool (System.CommandLine)
│   ├── Graphify.Sdk/            # GitHub Copilot SDK integration
│   ├── Graphify.Mcp/            # MCP stdio server
│   └── tests/
│       ├── Graphify.Tests/              # Unit tests (xUnit)
│       └── Graphify.Integration.Tests/  # Integration tests
└── .github/workflows/           # CI/CD workflows
```

- **Root files only:** README.md, LICENSE, Directory.Build.props, global.json, .gitignore, graphify-dotnet.slnx
- **All source code** (libraries, tests, CLI, servers) lives under `src/`
- No `images/`, `docs/`, or `samples/` directories initially

---

## ⚠️ Critical Rule: All Code Lives Under `src/`

**Every** project — libraries, tests, tools, servers — MUST be placed under the `src/` directory. Nothing gets created at the repo root except:
- README.md
- LICENSE
- Directory.Build.props
- global.json
- .gitignore
- .editorconfig
- graphify-dotnet.slnx

**Structure under `src/`:**
- `src/Graphify/` — Core library (graph pipeline, data structures, export)
- `src/Graphify.Cli/` — CLI tool (dotnet tool, System.CommandLine)
- `src/Graphify.Sdk/` — GitHub Copilot SDK integration
- `src/Graphify.Mcp/` — MCP stdio server (ModelContextProtocol)
- `src/tests/Graphify.Tests/` — Unit tests (xUnit)
- `src/tests/Graphify.Integration.Tests/` — Integration tests

**Golden rule: NEVER create project directories at the repo root. NEVER put tests or tools outside `src/`.**

---

## Solution Format

- Use `.slnx` (XML-based solution format), **not** `.sln`
- All projects (libraries, tests, CLI, servers) must be included in the solution file
- Use `<Folder>` elements for logical grouping: `/src/`, `/src/tests/`

---

## Project Conventions

### Library / Application Projects
- Target `net10.0` only (single target — this is a .NET 10 project, not multi-target)
- Project naming: `Graphify`, `Graphify.Cli`, `Graphify.Sdk`, `Graphify.Mcp`
- Enable deterministic CI builds: `<ContinuousIntegrationBuild Condition="'$(GITHUB_ACTIONS)' == 'true'">true</ContinuousIntegrationBuild>`
- Include `<InternalsVisibleTo>` for the corresponding test project where needed
- `Graphify.Cli` includes `<PackAsTool>true</PackAsTool>` for NuGet dotnet-tool publishing

### Test Projects
- Target `net10.0`
- Framework: xUnit with coverlet.collector
- Naming: `Graphify.Tests`, `Graphify.Integration.Tests`
- Location: `src/tests/{TestProject}/`
- Set `<IsPackable>false</IsPackable>` and `<IsTestProject>true</IsTestProject>`

---

## Directory.Build.props

Place at repo root. Shared by all projects. NuGet packaging properties for the CLI tool are in `Graphify.Cli.csproj`:

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
    <RepositoryUrl>https://github.com/elbruno/graphify-dotnet</RepositoryUrl>
    <RepositoryType>git</RepositoryType>
  </PropertyGroup>
</Project>
```

---

## global.json

Target .NET 10 with `"rollForward": "latestMajor"` for flexibility:

```json
{
  "sdk": {
    "version": "10.0.100",
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
- **SDK:** `dotnet-version: 10.0.x`
- **Commands use solution-level operations** (not per-project)
- **publish.yml** handles NuGet publishing on release events

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

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [elbruno/graphify-dotnet](https://github.com/elbruno/graphify-dotnet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-28 -->
