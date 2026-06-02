---
trigger: always_on
description: This document serves as a guide for AI agents interacting with the Melodee solution. It outlines the project structure, coding standards, and available resources to ensure consistent and high-quality contributions.
---

## Melodee AI Agent Guide

### Overview
This document serves as a guide for AI agents interacting with the Melodee solution. It outlines the project structure, coding standards, and available resources to ensure consistent and high-quality contributions.

### Project Context
Melodee is a comprehensive music and media management system built with .NET (C#) and Blazor. It includes features like Party Mode, Jukebox functionality, and media library management.

### AI Resources
This repository contains a suite of configuration files designed to guide AI behavior, located in the [`.github/`](./.github/) directory.

### Quick start: pick the right instruction set
Before editing, open the most relevant instruction file(s) under [`.github/instructions/`](./.github/instructions/).

- **Security-sensitive changes** (auth, tokens, cookies, file paths, external URLs, anything user-supplied): [`security-and-owasp.instructions.md`](./.github/instructions/security-and-owasp.instructions.md)
- **Performance-sensitive changes** (hot paths, DB queries, streaming, large collections): [`performance-optimization.instructions.md`](./.github/instructions/performance-optimization.instructions.md)
- **Code review output** (reviewing PRs/patches): [`code-review-generic.instructions.md`](./.github/instructions/code-review-generic.instructions.md)
- **Docs** (any `*.md`): [`markdown.instructions.md`](./.github/instructions/markdown.instructions.md)
- **Playwright tests**:
  - [.NET](./.github/instructions/playwright-dotnet.instructions.md)
  - [TypeScript](./.github/instructions/playwright-typescript.instructions.md)
  - [Python](./.github/instructions/playwright-python.instructions.md)

### 1. Custom Instructions (`.github/instructions/`)
These files define the specific coding standards, architectural guidelines, and best practices for the project. Agents **MUST** adhere to these instructions when working on relevant parts of the codebase.

See: [`.github/instructions/`](./.github/instructions/)

- **ASP.NET & Blazor**:
  - [`aspnet-rest-apis.instructions.md`](./.github/instructions/aspnet-rest-apis.instructions.md)
  - [`blazor.instructions.md`](./.github/instructions/blazor.instructions.md)
  - [`blazor-localization.instructions.md`](./.github/instructions/blazor-localization.instructions.md)
- **Languages**:
  - [`csharp.instructions.md`](./.github/instructions/csharp.instructions.md)
  - [`python.instructions.md`](./.github/instructions/python.instructions.md)
  - [`shell.instructions.md`](./.github/instructions/shell.instructions.md)
  - [`yaml.instructions.md`](./.github/instructions/yaml.instructions.md)
  - [`markdown.instructions.md`](./.github/instructions/markdown.instructions.md)
- **Data & ORM**:
  - [`ef-core-migrations.instructions.md`](./.github/instructions/ef-core-migrations.instructions.md)
  - [`nodatime.instructions.md`](./.github/instructions/nodatime.instructions.md)
- **Quality & Testing**:
  - [`testing.instructions.md`](./.github/instructions/testing.instructions.md)
  - [`playwright-dotnet.instructions.md`](./.github/instructions/playwright-dotnet.instructions.md)
  - [`playwright-typescript.instructions.md`](./.github/instructions/playwright-typescript.instructions.md)
  - [`playwright-python.instructions.md`](./.github/instructions/playwright-python.instructions.md)
  - [`code-review-generic.instructions.md`](./.github/instructions/code-review-generic.instructions.md)
- **Architecture & Best Practices**:
  - [`dependency-injection.instructions.md`](./.github/instructions/dependency-injection.instructions.md)
  - [`dotnet-architecture-good-practices.instructions.md`](./.github/instructions/dotnet-architecture-good-practices.instructions.md)
  - [`performance-optimization.instructions.md`](./.github/instructions/performance-optimization.instructions.md)
  - [`security-and-owasp.instructions.md`](./.github/instructions/security-and-owasp.instructions.md)
  - [`self-explanatory-code-commenting.instructions.md`](./.github/instructions/self-explanatory-code-commenting.instructions.md)
  - [`task-implementation.instructions.md`](./.github/instructions/task-implementation.instructions.md)
  - [`github-actions-ci-cd-best-practices.instructions.md`](./.github/instructions/github-actions-ci-cd-best-practices.instructions.md)
- **Infrastructure**:
  - [`docker.instructions.md`](./.github/instructions/docker.instructions.md)

### 2. Agent Personas (`.github/agents/`)
Specialized agent definitions for specific tasks. Use these personas to adopt the appropriate mindset and toolset.

See: [`.github/agents/`](./.github/agents/)

- [`expert-dotnet-software-engineer.agent.md`](./.github/agents/expert-dotnet-software-engineer.agent.md): General purpose high-level .NET engineering.
- [`csharp-dotnet-janitor.agent.md`](./.github/agents/csharp-dotnet-janitor.agent.md): Cleanup and maintenance.
- [`debug.agent.md`](./.github/agents/debug.agent.md): Dedicated debugging specialist.
- [`dotnet-upgrade.agent.md`](./.github/agents/dotnet-upgrade.agent.md): For handling framework upgrades.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [melodee-project/melodee](https://github.com/melodee-project/melodee) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-02 -->
