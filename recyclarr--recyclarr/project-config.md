---
trigger: always_on
description: enables the user-visible capability.
---

# AGENTS

.NET CLI tool for synchronizing TRaSH Guides to Sonarr/Radarr. Mainline branch: `master`.

## Linear

- Issue prefix: `REC-` (e.g. REC-74 is Linear; #74 is GitHub)
- Issue statuses: Backlog, Todo, In Progress, Done, Canceled, Duplicate
- Project statuses: Backlog, Planned, In Progress, Completed, Canceled
- Labels: Bug, Tech Debt, Documentation, Templates, Blocked By Trash Guides

Issue lifecycle:

- MUST transition to "In Progress" when starting work
- MUST transition to "Done" after the final commit lands
- MUST check comments when reading issue details

Project lifecycle:

- MUST transition to "In Progress" when starting work on any issue in the project
- MUST transition to "Completed" when all issues are done
- SHOULD transition to "Canceled" if the project is abandoned

Scope separation:

- Linear tracks work items and progress; the repo tracks design and architectural knowledge
  (`docs/architecture/`, `docs/decisions/`). MUST NOT duplicate between them.
- Linear project descriptions SHOULD link to relevant ADRs and architecture docs.
- MUST NOT put design rationale, decision records, or living design reference into Linear.

## Agent Architecture

Single primary agent with direct access to all files and tools. Subagents for bounded contexts:

- **trash-guides**: Read-only research against the TRaSH Guides repo (uses haiku for cost). MUST use
  this subagent for any question about TRaSH Guides content (custom formats, quality profiles,
  naming, quality sizes, trash_ids). NEVER use the generic explore agent for guides research.
- **commit**: Git operations after user approval

## Skills

Per-skill triggers. MUST load before acting on the governed task; a skill loaded in parallel with
that action arrives too late.

- `testing`: MUST load for any work under `tests/**`, including authoring tests, updating E2E
  fixtures, debugging failures, or running `coverage.py` / `Run-E2ETests.ps1`.
- `changelog`: MUST load when adding, editing, or reorganizing entries in `CHANGELOG.md`, or when
  drafting release notes.
- `decisions`: MUST load when creating, editing, or superseding ADRs or PDRs under
  `docs/decisions/`.
- `mapperly`: MUST load when writing, editing, or debugging `Riok.Mapperly` mapper classes
  (`[Mapper]`-attributed partials, `*Mapper.cs` under `ServarrApi/`, RMG-prefixed diagnostics).
- `duplication-vs-abstraction`: MUST load when weighing whether to extract a shared abstraction,
  base class, or generic helper, particularly across Sonarr/Radarr parallels, Refit-generated
  clients, or anti-corruption layers over distinct external systems.
- `rx-observables`: MUST load when writing, editing, or reviewing code that uses `System.Reactive`
  (Rx.NET), `IObservable`/`IObserver`, subjects, `CompositeDisposable`, or `TestScheduler`.
- `minimal-apis`: MUST load when writing, editing, or reviewing ASP.NET Core Minimal API endpoints,
  route handlers, route groups, endpoint filters, OpenAPI configuration, or embedded Kestrel server
  code.

## Project Context

- Uses SLNX format (`Recyclarr.slnx`) instead of traditional SLN files.
- Components: Cli (entry) -> Core (logic) -> TrashGuide/ServarrApi (integrations)
- DI: Autofac. Every library gets its own Autofac Module to keep registration modular.
- Config: YAML with JSON Schema validation (see YAML Schema Maintenance)
- Testing: NUnit 4 + NSubstitute + AutoFixture + parallel execution
- Dotnet tools in `.config/dotnet-tools.json`
- CLI: `Spectre.Console` package for CLI framework

## Code Review Comments

The `// CodeReview:` marker flags questions or concerns for review before commit. A pre-commit hook
prevents accidental commits containing these markers.

Lifecycle:

1. Human adds `// CodeReview: <question>` during implementation
2. Agents MUST stop and present each unresolved CodeReview comment to the user before declaring work
   complete
3. Resolution: either address the concern (refactor, add context) or the user explicitly dismisses
   it
4. Remove the marker only after resolution; NEVER silently delete

When running `pre-commit run` mid-development (before commit), pass `SKIP=no-review-markers` to
suppress the hook: `SKIP=no-review-markers pre-commit run --files <files>`

## Coding Standards & Development Requirements

- You MUST use dependency injection for all dependencies; NEVER manually 'new' objects in production
  code. Concrete implementations get injected; tests can substitute. Search existing registrations
  before adding new ones.
- Search existing code first: `rg "pattern"` before writing new code. Holistically and
  comprehensively make changes, don't just do it in isolation which ignores other important areas of
  code that might be in-scope or indirectly affected by a change.
- Reuse or extend existing implementations before adding parallel ones. DRY targets knowledge
  duplication, not incidental syntactic similarity; when weighing extraction of a shared abstraction
  (base class, generic helper, unified interface), load the `duplication-vs-abstraction` skill
  first.
- CRITICAL: Follow SOLID, YAGNI principles. Every abstraction must justify its existence with
  concrete current needs.
- .NET 10.0 (C# 14) + nullable reference types
- Zero warnings/analysis issues — treat warnings as errors

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [recyclarr/recyclarr](https://github.com/recyclarr/recyclarr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
