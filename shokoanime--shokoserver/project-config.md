---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
dotnet build Shoko.Server.sln
dotnet test Shoko.Tests/Shoko.Tests.csproj --filter "FullyQualifiedName~ClassName.Method"
dotnet test Shoko.IntegrationTests/Shoko.IntegrationTests.csproj
```

Target framework: `.NET 10.0`. Configurations: `Debug`, `Release`, `ApiLogging`, `Benchmarks` (server + benchmarks only; tests use `Debug`/`Release`).

## Code Style

`.editorconfig` with ReSharper enforcement:
- Line length: 160 characters
- Modifier order: `private, protected, public, internal, sealed, new, override, virtual, abstract, static, extern, async, unsafe, volatile, readonly, required, file`
- **`var` preferred everywhere** — `csharp_style_var_elsewhere`, `csharp_style_var_for_built_in_types`, and `csharp_style_var_when_type_is_apparent` all set to `true` (enforced as warnings in `src/` paths)
- Braces on new lines (`csharp_new_line_before_open_brace = all`)
- Naming: `_camelCase` for instance fields, `_camelCase` for static fields, `PascalCase` for methods/classes/properties, `camelCase` for locals/parameters

## Commit Messages

We follow [Conventional Commits](https://www.conventionalcommits.org/), formatted as `type(scope): subject` (the scope is optional).

- **Subject** — present/imperative tense, lowercase, no trailing period (e.g. `add stream indices`, not `Added stream indices.`).
- **Body** — past tense, describing what the commit did. Optional; include it when the *why* isn't obvious from the subject. Markdown is allowed, but **without headers** — use bold text and bullet lists instead of `#` headings.
- **Code references** — wrap code in backticks in both the subject and body: methods (`` `FindReleaseForVideo` ``), types (`` `VideoLocal` ``), endpoints/paths (`` `/api/v3/Series` ``), settings, and file names. Use the bare identifier, not a prose description.
- Append `[skip ci]` to the subject for changes that should not trigger CI.

### Types

| Type | When to use |
|------|-------------|
| `feat` | Adds a new user-, API-, or plugin-facing capability (endpoint, DTO field, abstraction surface). |
| `fix` | Corrects broken or incorrect behavior. |
| `refactor` | Restructures code without changing external behavior — renames, splits, simplifies, removes. |
| `chore` | In-tree housekeeping with no behavior change — version bumps, import sorting, comment/doc-comment fixups. |
| `docs` | Documentation-only changes — XML docs, READMEs, API deprecation notices. |
| `repo` | Repository infrastructure and tooling — workflows, scripts, dependencies, devcontainer, `.gitignore`/`.dockerignore`. |
| `misc` | Small changes that don't fit cleanly elsewhere. Reach for this only when nothing else fits. |
| `revert` | Reverts a previous commit. |

### Precedence

When a commit qualifies for more than one type, pick the most structural one. Dominance order: `refactor` > `feat` > `fix` > `chore`/`repo`/`docs`/`misc`.

- A commit that both adds a feature **and** restructures or removes existing code is a `refactor`, not a `feat`.
- Introducing a new feature that requires dropping a previous feature is a `refactor` (the removal dominates).

### Disambiguation

- **`chore` vs `repo`** — `chore` is in-tree code housekeeping (imports, version bumps, comments); `repo` is the build/CI/tooling/deps that live around the code.
- **`refactor` vs `fix`** — if external behavior visibly changes for the better, it's `fix`; if behavior is identical, it's `refactor`.
- **`chore` vs `misc`** — prefer a precise type; `misc` is the last resort.
- **`docker` is a scope, not a type** (e.g. `repo(docker)`, `feat(docker)`).

### Scopes

Scopes are optional and free-form, but reuse the established ones where they apply (non-exhaustive): `abstractions`, `api`, `db`, `plugin`, `images`, `relocation`, `anidb`, `tmdb`, `search`, `scrobble`, `core`, `deps`, `workflows`, `scripts`, `docker`.

### Example

```
refactor: remove parallel mode and simplify release search pipeline

Collapsed the dual-path release search into a single sequential
pipeline. The parallel evaluator added contention without a measurable
throughput gain on real libraries.

- Removed the parallel branch and its bespoke locking
- Folded the remaining provider lookup into `FindReleaseForVideo`
```

## Architecture

### Project Layout

- **`Shoko.Abstractions`** — NuGet package for plugin authors. Defines the interface contract between the core and plugins (`IPlugin`, `IShokoSeries`, `IShokoEpisode`, `IVideo`, `IUser`, and all service/metadata/video/user interfaces). Only update this when the plugin contract itself needs to change.
- **`Shoko.Server`** — All implementation: API, database, repositories, services, scheduling, providers, models.
- **`Shoko.QueueProcessor`** — Custom job queue engine. Defines `IQueueScheduler`, `IQueueJob`, `RecurringJobRegistry`, `IJobChainBuilder`, `IVideoReleaseProviderJob<T>`, persistence (`QueuedJob`, `JobRepository` via EF Core), concurrency/acquisition attributes, and the orchestration stack (`QueueOrchestrator`, `WorkerPool`, `WorkerPoolManager`). Referenced as a local project, not a NuGet package.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ShokoAnime/ShokoServer](https://github.com/ShokoAnime/ShokoServer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
