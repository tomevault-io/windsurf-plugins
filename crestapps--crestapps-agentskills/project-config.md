---
trigger: always_on
description: **ALWAYS reference these instructions first and fall back to searching only if needed.**
---

# CrestApps.AgentSkills Development Instructions

**ALWAYS reference these instructions first and fall back to searching only if needed.**

## Project Overview

CrestApps.AgentSkills contains shared AI agent skills and MCP tooling for .NET applications and Orchard Core projects.

- **Target Framework**: .NET 10 (net10.0)
- **SDK Version**: .NET 10.0.100 (see `global.json`)
- **Package Management**: Central Package Management via `Directory.Packages.props`
- **Skill source of truth**: `src/CrestApps.AgentSkills/orchardcore/` (106 Orchard Core skills)

### Four Source Projects

1. **`CrestApps.AgentSkills`** - Skill content container
   - Contains all Orchard Core skill files under `orchardcore/`
   - Not packable — purely a source container referenced by the other packages

2. **`CrestApps.AgentSkills.Mcp`** - Generic MCP engine
   - Framework-agnostic skill parser and MCP provider
   - Supports `.md` (front-matter) and `.yaml`/`.yml` skill formats
   - Key interfaces: `IAgentSkillFilesStore`, `IMcpPromptProvider`, `IMcpResourceProvider`
   - Key implementations: `DefaultAgentSkillFilesStore`, `SkillPromptProvider`, `SkillResourceProvider`
   - Key parsers: `SkillFrontMatterParser`, `SkillYamlParser`, `SkillFileParser`
   - No bundled skills — expects consumer to provide skill directory

3. **`CrestApps.AgentSkills.OrchardCore`** - Dev-time skill distributor
   - Development dependency only (`IncludeBuildOutput=false`, `DevelopmentDependency=true`)
   - Uses MSBuild `.targets` to copy skills to solution root `.agents/skills/` on first build
   - No runtime code — purely for local AI authoring

4. **`CrestApps.AgentSkills.Mcp.OrchardCore`** - Runtime MCP server for Orchard Core
   - Extends `CrestApps.AgentSkills.Mcp` with Orchard Core–specific convenience wrappers
   - Bundles skills via `contentFiles` (copied to bin output on restore)
   - Entry point: `OrchardCoreSkillMcpExtensions` (`AddOrchardCoreSkills`, `AddOrchardCoreAgentSkillServices`)
   - Services registered as singletons for caching

**Tests**: `test/CrestApps.AgentSkills.Mcp.Tests/`, `test/CrestApps.AgentSkills.Mcp.OrchardCore.Tests/`

## Build & Test

Run builds and tests from the repository root:

```bash
# Build (treat warnings as errors, run analyzers)
dotnet build -c Release -warnaserror /p:TreatWarningsAsErrors=true /p:RunAnalyzers=true /p:NuGetAudit=false

# Run all tests
dotnet test -c Release --no-build --verbosity normal

# Run tests from a single project
dotnet test -c Release --no-build test/CrestApps.AgentSkills.Mcp.Tests/

# Run a specific test class
dotnet test -c Release --no-build --filter "FullyQualifiedName~SkillFrontMatterParserTests"

# Run a specific test method
dotnet test -c Release --no-build --filter "FullyQualifiedName~SkillFrontMatterParserTests.TryParse_ValidFrontMatter_ReturnsTrueAndExtractsFields"
```

**Note**: Tests use xUnit v3 (`xunit.v3`). Test classes follow the pattern `<Subject>Tests.cs` with the `sealed` modifier.

## Skill Validation

Each skill directory under `src/CrestApps.AgentSkills/orchardcore/` must contain a `SKILL.md` file with YAML front-matter (must include `name` and `description`).

### Skill Requirements

- **File name**: `SKILL.md` (uppercase, not `skill.md`)
- **Directory naming**: lowercase, hyphenated, prefixed with `orchardcore-` (e.g., `orchardcore-content-types`)
- **`name` field**: Must exactly match the directory name (e.g., `orchardcore-content-types`)
- **Front-matter**: Must start with `---` and contain closing `---`
- **References**: Optional `references/` subdirectory for additional `.md` files (not `examples/`)
- **Description length**: Keep `description` under the `1024` character limit

### Front-Matter Safety Rules

- The `description` field is intentionally kept as a plain single-line YAML scalar in this repository unless quoting is absolutely necessary.
- **Do not introduce raw `: ` inside an unquoted `description` value.** Phrases like `Step 1: Build`, `Example: Foo`, `Client: SSE`, or `Workflow: Publish` can break `skills-ref` YAML parsing.
- Rewrite those phrases instead, for example:
  - `Step 1 Build`
  - `Example Foo`
  - `MCP Client Connecting to External MCP Servers`
  - `Two Approaches Webhook vs. Protocol-Agnostic Relay`
- Watch for namespaced configuration keys in descriptions as well. If mentioned in front matter, prefer forms that avoid YAML-like `key: value` text.
- After editing any skill front matter, run a quick repository-wide search for unsafe descriptions, for example searching `^description: .*: .*` across `src/CrestApps.AgentSkills/orchardcore/**/SKILL.md`.
- If a description truly requires YAML quoting to stay correct, keep it valid YAML first, but prefer rewording over quoting when possible to stay consistent with the existing corpus and user preference.

### Skill Documentation Conventions (from CONTRIBUTING.md)

- All recipe step JSON blocks must be wrapped in root recipe format: `{ "steps": [...] }`
- All C# classes in code samples must use the `sealed` modifier, **except for View Models** which must not be `sealed` because they are used for model binding
- Third-party module packages (non `OrchardCore.*`) must be installed in the web/startup project
- Keep guidance concise, example-driven, and actionable

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CrestApps/CrestApps.AgentSkills](https://github.com/CrestApps/CrestApps.AgentSkills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
