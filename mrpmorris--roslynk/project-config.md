---
trigger: always_on
description: This is a source-based implementation guide for future changes. Paths below are relative to the repository root unless a section says otherwise. Re-check the affected implementation before changing it; this guide describes the architecture and its current limits, not a substitute for the source.
---

# Working on Roslynk

This is a source-based implementation guide for future changes. Paths below are relative to the repository root unless a section says otherwise. Re-check the affected implementation before changing it; this guide describes the architecture and its current limits, not a substitute for the source.

## Start here

- The solution is `Source/Morris.Roslynk.slnx` (not under `Source/App`).
- Read `git status` before editing and preserve unrelated work.
- When Roslynk is connected, use its semantic tools for compiled C#, Razor and CSHTML: open the solution, use `get_members`/`get_symbol_body` to inspect implementations, and use reference/caller queries for impact analysis. Follow `skills/roslynk/SKILL.md` and `skills/roslynk/references/tools.md` for tool usage. Retry `Indexing` while loading; do not manually reload for ordinary source edits.
- Batch related read-only questions with `multi_query`. Re-query after writes. A running Roslynk daemon can inspect edited source, but editing this repository does **not** replace the daemon's executing implementation; run tests against the checkout to validate new behavior.
- Use normal file tooling for non-source configuration, documentation, and new-file creation. Roslynk's patch tool only edits existing files, and its plain-text fallback is rooted at the solution directory (`Source`), not the repository root.
- The most important seams are `RoslynInstance`, `SolutionModel`, `ApplyPipeline`, `AtomicFileWriter`, `SolutionFileSync`, and the internal cores used by `MultiQueryCatalog`.

## Required workflow for feature and tool changes

- When creating a new read-only tool, assess whether it is a candidate for `multi_query` and **ask the user whether to include it**. Do not silently include or exclude it. Continue independent implementation work while that choice is pending; if included, implement the pinned-model core, catalog/enum integration and relevant tests described below.
- Add every new feature and feature change to the `# Unreleased` section at the top of `releases.md`. Create that section at the top if it is missing, preserving existing release history.
- Always keep consumer-facing skill and tool documentation synchronized with tool definitions and their descriptions, including parameters, defaults, output, errors, supported scope and batch availability. In this repository the files are `skills/roslynk/SKILL.md` (singular `SKILL.md`, the consumer skill file) and `skills/roslynk/references/tools.md`. Update both as applicable to the changed contract; checking their consistency is part of completing every tool change. Keep any additional consumer `SKILLS.md`/`tools.md` copies synchronized if introduced later.

## Repository map and build conventions

| Location | Responsibility |
| --- | --- |
| `Source/App/Morris.Roslynk` | Engine, feature tools, shared infrastructure, DI and MCP tool registration. |
| `Source/App/Morris.Roslynk.Mcp` | ASP.NET Core host, loopback HTTP transport, stdio bridge/daemon startup, idle eviction and OpenTelemetry integration. Ships as the `Roslynk` .NET tool, command `roslynk`. |
| `Source/App/Morris.Roslynk.AppHost` | Aspire development host. |
| `Source/App/Morris.Roslynk.Tests` | Engine, feature, concurrency, file synchronization and writing tests. |
| `Source/App/Morris.Roslynk.McpTests` | Host composition, published MCP schemas, tool invocation and transport-facing tests. |
| `Source/TestFixtures` | Small independent solutions: Simple, Broken, References, Conditional, Razor, Cshtml, LocalFunction, CodeStyle and Generator. |
| `skills/roslynk` | User-facing agent skill and detailed tool contracts. Update when tool behavior changes. |
| `README.md`, `releases.md` | Setup/tool guidance and release history. |

`Source/Directory.Build.props` sets .NET 10, latest C#, nullable and implicit usings, and warnings as errors. Package versions are central in `Source/Directory.Packages.props`; do not add local versions to individual project references. `Source/.editorconfig` specifies tabs (width 4), Allman braces, file-scoped namespaces, CRLF for C#/VB, and usings outside namespaces. Existing classes commonly use PascalCase private fields and explicit constructor injection; follow nearby code without unrelated formatting churn.

Feature tools live in `Features/<Area>/<Operation>/<Operation>Tool.cs` (MultiQuery is a flatter slice). Shared mechanisms live under `Infrastructure`. Namespace structure follows directories. Both test assemblies have access to core internals via `InternalsVisibleTo`.

## Lifecycle, snapshots and ordering

Sources: `Infrastructure/Lifecycle/{InstanceRegistry,RoslynInstance,SolutionModel,WriteResult,SemaphoreSlimReadWriteLock}.cs` under the core project.

`InstanceRegistry` owns a concurrent dictionary of lazy `RoslynInstance`s keyed by normalized solution paths (`SolutionKey`). Key comparison is case-insensitive on Windows/macOS, ordinal on other platforms. Do not invent a second registry or cache keyed by an unnormalized path.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mrpmorris/Roslynk](https://github.com/mrpmorris/Roslynk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
