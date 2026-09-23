---
trigger: always_on
description: This repository maintains **Dapper.FluentMap**, a public multi-package .NET library that provides fluent, strongly typed mapping between POCO properties and database columns used by Dapper, keeping persistence attributes out of domain models.
---

# AGENTS.md

## Purpose

This repository maintains **Dapper.FluentMap**, a public multi-package .NET library that provides fluent, strongly typed mapping between POCO properties and database columns used by Dapper, keeping persistence attributes out of domain models.

Agent work must be small, correct, reproducible, and compatible with existing public behavior. Respond in Portuguese unless the user explicitly asks otherwise.

## Non-Negotiable Rules

| Area | Rule |
| --- | --- |
| Branching | The default development and release branch is `master`. Never modify `master` directly; create or use a task branch. |
| Scope | Prefer the smallest cohesive change. Do not mix functional change, modernization, dependency updates, refactoring, and release work unless the task explicitly spans them. |
| Public library | Preserve source, binary, and behavioral compatibility unless a breaking change is explicitly requested, justified, tested, documented, and versioned. |
| Publishing | Do not publish packages, create tags, create GitHub Releases, or run release/recovery workflows unless explicitly requested. |
| Secrets | Never commit secrets, tokens, API keys, certificates, or credential material. |
| Identity | Project name, project path, assembly name, C# namespace, and NuGet `PackageId` are independent identities. Never rename one merely because another changes; determine explicitly which identity the task intends to change. |
| Targets | Public packages currently preserve `netstandard2.0`. Do not change targets, multi-targeting, SDK, nullable, AOT, analyzers, test framework, or the repository's dependency-versioning model as incidental work. |
| Dapper | Use public Dapper contracts only. Do not copy Dapper internals or assume unit metadata tests prove end-to-end Dapper materialization. |

## Repository Map

| Path | Role |
| --- | --- |
| `src/Dapper.FluentMap/` | Core package and default production scope. |
| `src/Dapper.FluentMap.Dommel/` | Optional Dommel integration; change only when requested or demonstrably required by a core contract change. |
| `src/Dapper.FluentMap.DependencyInjection/` | Optional DI integration. |
| `src/Dapper.FluentMap.Analyzers/` | Roslyn analyzer package. |
| `src/Dapper.FluentMap.Generators/` | Source generator package. |
| `test/**` | Unit, integration, provider, analyzer/generator, DI, generated-registration, and AOT smoke tests. |
| `benchmarks/**` | BenchmarkDotNet benchmarks; not part of ordinary validation unless the task concerns performance. |
| `eng/**` | Validation, release, rollback, Sonar, package-catalog, and consumer-smoke helper scripts. |
| `.github/workflows/**` | CI, release, release recovery, and Sonar automation. |
| `.agents/skills/**` | Repository-local agent skills; load only those relevant to the current task. |

Prefer `Dapper.FluentMap.slnx` for current SDK workflows; `Dapper.FluentMap.sln` remains a compatibility fallback.

## Sources Of Truth

Read only what the task needs, in this order:

1. `AGENTS.md`
2. Relevant `.agents/skills/*/SKILL.md`
3. User request, issue, PR, or diff
4. `README.md`, `COMPATIBILITY.md`, `MIGRATION.md`, `CHANGELOG.md` when behavior or compatibility is involved
5. `Dapper.FluentMap.slnx` / `Dapper.FluentMap.sln`
6. Relevant `.csproj`, `Directory.Build.props`, `Directory.Build.targets`, `global.json`, `NuGet.Config`
7. Relevant source and test files
8. Relevant workflows and `eng` scripts when CI, packaging, or release is involved

Do not load the whole repository by default. Search first with `rg` / `rg --files`, then open the narrow files that define the contract, implementation, tests, or automation being changed.

## Skill Routing

Read `AGENTS.md` first. Load only the skills relevant to the current task; do not load every skill by default. Multiple skills may be combined when the task genuinely spans multiple concerns. Repository-specific rules in this file take precedence over generic guidance in a skill. Skills must inspect the current repository rather than assuming their source-template baseline still applies.

| Task | Primary skill |
| --- | --- |
| Implement a well-defined issue | `dotnet-issue-implementation` |
| Production/library/project/PackageId change | `dotnet-library-change` |
| Behavior-preserving refactoring | `dotnet-refactoring-engineer` |
| Pull request/diff review | `dotnet-pr-review` |
| CI, packaging, NuGet, versioning, release or recovery semantics | `ci-release-governance` |
| GitHub Actions YAML authoring/structural validation | `authoring-github-workflows` |
| NuGet.org OIDC/trusted-publishing review or diagnosis | `nuget-trusted-publishing` |
| Find caller-visible behaviors existing tests would miss | `test-gap-analysis` |
| BenchmarkDotNet/performance comparison | `microbenchmarking` |
| `Directory.Build.*` / MSBuild organization | `directory-build-organization` |
| Diagnose unclear MSBuild failures from `.binlog` | `binlog-failure-analysis` |

Pair skills when concerns overlap. Examples:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rodri-oliveira-dev/Dapper-FluentMap](https://github.com/rodri-oliveira-dev/Dapper-FluentMap) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
