---
trigger: always_on
description: FlatRedBall2 is a 2D game engine/framework written in C# on .NET, built on top of MonoGame. It integrates Gum (UI) and Tiled (level editing) as dependencies.
---

@design/TODOS.md

# Repository Guidelines

## What Is This?

FlatRedBall2 is a 2D game engine/framework written in C# on .NET, built on top of MonoGame. It integrates Gum (UI) and Tiled (level editing) as dependencies.

## Prerelease Status — Breaking Changes Are Free

**FlatRedBall2 is in active prerelease development. There are no shipping consumers, no public API surface to preserve, and no backwards-compatibility obligations.** Anything in the engine — type names, method signatures, namespaces, defaults, semantics — can be changed at any time if a better design exists.

When proposing or evaluating a change:
- **Do not** raise "this will break existing code" as a concern. It is not a concern. Existing call sites in samples and tests are part of this codebase and will be updated alongside the change.
- **Do not** add deprecation shims, `[Obsolete]` attributes, alias members, or "kept for compatibility" wrappers. Just change the thing.
- **Do** propose the cleanest API; if a rename or restructuring makes the engine clearer, it is on the table.

The only relevant question is "is this the right design?" — never "will this break someone?"

## Key Files

- Main project: `src/FlatRedBall2.csproj` (MonoGame.Framework.DesktopGL, version pinned in the root `Directory.Packages.props`)
- Code style: `.claude/code-style.md`
- Deferred items: `design/TODOS.md`
- Multi-phase plans: `plan/plan.md` (index) — see below
- Test project: `tests/FlatRedBall2.Tests/FlatRedBall2.Tests.csproj`

## Multi-Phase Plans (`plan/`)

Work too large for a single PR gets a **plan** rather than a long-running branch or a TODO entry.

- **`plan/plan.md` is the table of contents.** One row per phase, linking to that phase's document,
  with a status. It never holds phase content itself. Read it first when picking up large work.
- **Phase documents live in `plan/<issue#>-<initiative-slug>/phase-NN-<slug>.md`.** Each is
  self-contained: the issue restated, the high-level proposed resolution, features/stories, and
  every step as a checkbox. A reader should not need the GitHub issue open to work the phase.
- **Write the next phase doc when the previous phase is stable**, not all up front — plans written
  before any code exists are wrong by the time they are read.
- **Check boxes off as work lands**, and update the status row in `plan/plan.md`. Add discovered
  work as new checkboxes rather than silently widening an existing one.

Do not put small actionable items here — those go in `design/TODOS.md`. Do not put single-subsystem
design write-ups here — those go in `design/*.md`. Game design documents stay in `.claude/designs/`.

## Build & Test

```
dotnet build src/FlatRedBall2.csproj
dotnet test tests/FlatRedBall2.Tests/
```

## Available Skills

Skills live in two locations, by audience:
- **`/frb-skills/`** — 3rd-party skills for game developers using FlatRedBall2 (entities, collision, physics, animation, etc.). These are the public, distributable skill set.
- **`/.claude/skills/`** — 1st-party skills for engine contributors only (TDD discipline, skill authoring, sample-project bootstrap, content-boundary philosophy, orchestrator).

The 3rd-party skills are also surfaced under `/.claude/skills/<name>` via local symlinks (gitignored) so Claude Code's auto-discovery picks them up while working on the engine. Edit the canonical copy under `/frb-skills/`.

Invoke these with the Skill tool when working on specific topics:
- `entities-and-factories` — Entity lifecycle, Add (shapes/Gum), Factory<T>, spawning
- `collision-relationships` — AddCollisionRelationship, move/bounce semantics
- `physics-and-movement` — Y+ up, gravity, Drag, GameRandom
- `timing` — Cooldown gates, repeating timers, entity lifetimes, FrameTime.DeltaSeconds
- `shapes` — AARect, Circle, Polygon, visual properties
- `input-system` — Keyboard, gamepad, input binding
- `camera` — Camera setup and transforms
- `screens` — Screen lifecycle and transitions
- `gumcli` — **Ask first** before any Gum UI code: use gumcli tool or code-only? Covers gumcli new, .csproj content includes, codegen
- `gum-integration` — UI with Gum (runtime usage; use `gumcli` skill first if user chose Gum tool)
- `gum-packaging` — Bundle a `.gumx` Gum project into a single `.gumpkg` (tar+brotli) for distribution; toggle loose-vs-bundle in csproj for diagnostics
- `content-and-assets` — Asset loading
- `content-hot-reload` — `Screen.WatchContent`, `ContentWatcher`, debouncing, in-place vs screen-restart decision
- `engine-overview` — **Start here.** What the engine does automatically, what game code must implement, what is stubbed, and critical gotchas
- `engine-tdd` — **Load before editing `src/`.** Failing test in `tests/FlatRedBall2.Tests/` required before any behavior change in `src/`
- `skill-creator` — **Load before editing any skill's `SKILL.md`.** Map-and-landmines philosophy, the 20% damped-growth rule (incl. landmine/fission exceptions), the deletion test, 8-line code-block cap, and shared-context budget discipline

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vchelaru/FlatRedBall2](https://github.com/vchelaru/FlatRedBall2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
