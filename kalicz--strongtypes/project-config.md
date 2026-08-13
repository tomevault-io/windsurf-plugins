---
trigger: always_on
description: This repository has multiple contributors. Do not rely on per-user memory,
---

# StrongTypes — Coding Conventions

## Audience

This repository has multiple contributors. Do not rely on per-user memory,
personal preferences, or prior-session context when working here. Every
convention that matters belongs in this file (or in the code itself) so that
every collaborator — human or AI — sees the same rules.

**Do not write to the external memory system** (`~/.claude/projects/…` or
any similar per-user scratch space). No project memories, no feedback
memories, no references — nothing. If something is worth remembering
about this repo, it goes in this file so it travels with the code and
every contributor sees it.

## C# conventions

- **Primary constructors** — use them wherever possible (classes, `DbContext`
  subclasses, test fixtures, etc.).

## Folder layout — feature slices

Organize code by feature, not by technical role. An extension, a JSON
converter, and the main type for the same feature live together in the
feature's folder:

- `src/StrongTypes/Strings/NonEmptyString.cs`
- `src/StrongTypes/Strings/NonEmptyStringExtensions.cs`
- `src/StrongTypes/Strings/NonEmptyStringJsonConverter.cs`
- `src/StrongTypes/Maybe/MaybeExtensions.cs`

Extensions that *produce* a feature type go into that feature's folder
even when the input type is from somewhere else (e.g. `ToMaybe` on `T?`
lives under `Maybe/`, because the result — not the receiver — is what
defines the slice).

Namespaces stay flat at `StrongTypes` regardless of folder nesting. Folder
structure is for humans; the namespace is one flat shelf. The same applies
to the test project — keep tests in `namespace StrongTypes.Tests` even
when they live in subfolders, to avoid shadowing type names (e.g., a
nested `StrongTypes.Tests.Maybe` namespace would hide the
`StrongTypes.Maybe<T>` type from sibling files).

## Validated types — the TryCreate / Create pattern

For types that enforce a validation rule (e.g., `NonEmptyString`,
`PositiveInt`), follow this factory pattern:

```csharp
// Returns null when validation fails. Caller handles the null case.
public static NonEmptyString? TryCreate(string? value) { ... }

// Throws when validation fails. Thin wrapper over TryCreate.
public static NonEmptyString Create(string? value)
    => TryCreate(value) ?? throw new ArgumentException(...);
```

Rules:
- The validation logic lives in `TryCreate` only. `Create` delegates.
- Constructors are `private` — callers must go through the factories.
- Use nullable reference types.

## Tests

All testing rules — unit, API / ASP.NET Core / OpenAPI integration,
configuration binding, WPF, and analyzer tests — live in
[`testing.md`](testing.md). **Read that file
before writing or modifying a test, and before writing any code that
will need tests** (a new strong type, a converter, an analyzer, an API
endpoint, …). It is the single source of truth; do not infer testing
conventions from existing tests without checking it first.

**One shared CRUD surface (don't reintroduce drift):** the API CRUD/PATCH
suite's create/get/update/PATCH scenarios live exactly once, in the abstract
`EntityCrudTestsBase`, so the scalar and interval wire shapes cannot drift. Its
two thin adapters — `EntityTests` (scalar wire) and `IntervalEntityTests`
(object wire) — add only wire-shape-specific pieces (bodies, the read assertion,
invalid-payload cases). A new shared scenario goes in `EntityCrudTestsBase`, not
in an adapter. See the "One shared CRUD surface" section of
[`testing.md`](testing.md) and the XML docs on `EntityCrudTestsBase` and its two
adapters.

## Skill — keep it in sync

The `Skill/` directory ships a Claude / Codex skill that teaches the
agent how to use this library in *consumer* projects (not this repo).
It's distributed as a release asset and lives entirely inside `Skill/`:

- `Skill/SKILL.md` — top-level guide, package table, decision trees,
  anti-patterns.
- `Skill/references/*.md` — per-feature references loaded on demand.

**Update it whenever the library's user-facing surface changes.** That
includes:

- Adding a new package — add a row to the `Packages` table in
  `SKILL.md`, add a row to "Helpers and integrations", and create a
  `Skill/references/<package>.md` covering when to use it, when not
  to use it, and the wiring snippet.
- Adding a new strong type — add a row to the appropriate type
  catalog table and either extend an existing reference or add a new
  one.
- Changing wiring (extension method names, registration calls) or
  package compatibility (target frameworks, dependency versions) —
  update the affected reference file *and* any `SKILL.md` snippet
  that mentions it.
- Changing the recommended pattern for something the skill calls out
  (decision trees, anti-patterns) — update the corresponding section
  of `SKILL.md`.

The skill must work standalone; it can't assume the reader has the
repository handy. Keep snippets self-contained and copy real examples
from passing tests where possible.

## StrongTypes.Api — purpose

An ASP.NET Core minimal API that exists purely as an integration-test
harness: it verifies that strong types round-trip correctly through the
ASP.NET Core request pipeline and EF Core against both SQL Server and
PostgreSQL (via Testcontainers). Every write endpoint persists to both

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [KaliCZ/StrongTypes](https://github.com/KaliCZ/StrongTypes) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
