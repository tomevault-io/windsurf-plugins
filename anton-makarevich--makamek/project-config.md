---
trigger: always_on
description: This file provides guidance to AI coding agents when working with code in this repository.
---

# AGENTS.md

This file provides guidance to AI coding agents when working with code in this repository.

## Overview

MakaMek is a cross-platform, turn-based tactical BattleTech implementation built with **.NET 10** and **AvaloniaUI**. It is inspired by [MegaMek](https://megamek.org/) but prioritizes simplicity and a mobile-/web-first UX. Runs on Windows, Linux, macOS, Web (WASM), Android, and iOS.

## Build & Test Commands

- **Build the whole solution:** `dotnet build MakaMek.slnx`
- **Run all tests:** `dotnet test MakaMek.slnx`
- **Run one test project:** `dotnet test tests/MakaMek.Core.Tests/MakaMek.Core.Tests.csproj`
- **Run a single test / filter:** `dotnet test tests/MakaMek.Core.Tests/MakaMek.Core.Tests.csproj --filter "FullyQualifiedName~SomeTestClass"`
- **Coverage (mirrors CI):**
  ```bash
  dotnet test tests/MakaMek.Core.Tests/MakaMek.Core.Tests.csproj \
    /p:CollectCoverage=true /p:CoverletOutputFormat=opencover \
    /p:ExcludeByAttribute=GeneratedCodeAttribute /p:Include=[Sanet.MakaMek.Core]*
  ```
  The coverage filter uses the source assembly name (test assembly name minus `.Tests`). See `skills/coverage-check`.
- **Diff-coverage (PR pipeline):** [Cocodif](https://github.com/sanet/Cocodif) runs as a composite GitHub Action in each coverage workflow. It parses the coverlet OpenCover XML, computes `git diff --merge-base` against the PR base branch, and posts a per-file diff-coverage report as a sticky PR comment (one per module). The action is informational only — no `fail-under` gate. Each module workflow passes its own `coverage.opencover.xml`, `include` globs scoped to `src/<Module>/**`, and a unique `comment-marker` so reports don't collide. For local diff-coverage, install the CLI: `dotnet tool install --global Sanet.Cocodif`.
- **Run the desktop app:** `dotnet run --project src/MakaMek.Avalonia/MakaMek.Avalonia.Desktop`

Assembly/root namespaces are prefixed `Sanet.` (e.g. `MakaMek.Core` → `Sanet.MakaMek.Core`), even though project/folder names omit it.

## Testing Conventions

Tests use **xUnit** + **Shouldly** (assertions) + **NSubstitute** (mocking). UI (Avalonia) is intentionally excluded from coverage; presentation logic lives in `MakaMek.Presentation` (ViewModels/UiStates) specifically so it *can* be unit-tested without the UI. Prefer adding logic there over the Avalonia layer.

## Versioning (required for PRs)

`Directory.Build.props` holds a single `<VersionPrefix>` for all packages. **Every PR that modifies files under `src/` must bump this version** — `pr-version-check.yml` fails the PR if the version is not greater than `main`. Bump it as part of your change.
- The version should only be incremented once per PR
- Test-only, docs-only, or infra-only PRs do not require a version bump.
- Agents may only bump the **patch** segment (e.g. `0.63.10` → `0.63.11`). Never change Major or Minor without explicit human approval.
- Commit messages follow Conventional Commits] (`feat`/`fix`/`docs`/`refactor`/`chore`/`build`, imperative subject).

## Architecture

The codebase is a layered set of projects; dependencies flow **Avalonia → Presentation → Core** (Core has no UI dependencies).

### Layers (`src/`)
- **MakaMek.Core** — Engine and all domain logic: game loop, state machine, phases, commands, units/components, combat & piloting mechanics, dice. No UI. This is where game rules live.
- **MakaMek.Map** — Hex-grid map representation, coordinates, terrain, map generation.
- **MakaMek.Presentation** — ViewModels and **UiStates** (per-phase interaction logic). The testable bridge between Core and the UI. Uses the `Sanet.MVVM` framework (see the `sanet-mvvm` skill before touching ViewModels/navigation/DI).
- **MakaMek.Avalonia** — AvaloniaUI views + per-platform heads (`.Desktop`, `.Android`, `.iOS`, `.Browser`) and shared `.Controls`.
- **MakaMek.Bots** — AI bot framework: per-phase decision engines, plus an experimental LLM-powered bot (agents/tools). Ships as a Docker BotAgent.
- **MakaMek.Services** — Platform-abstraction interfaces (files, images, dispatcher, PDF export) with Avalonia implementations in `MakaMek.Services.Avalonia`.
- **MakaMek.Assets** — 2D asset management (unit/terrain images).
- **MakaMek.Localization** — Localized strings.
- **MakaMek.SourceGenerators** — Roslyn generators that build type registries at compile time (command types, component providers, movement-cost / roll-modifier / PSR-context resolvers). If you add a new command/component/modifier type, the registry is generated — don't hand-maintain a switch.

### Core game model (the "big picture")

Client-server architecture even for local play (RX-based transport locally, SignalR for LAN):
- **`BaseGame`** is the abstract root. **`ServerGame`** owns authoritative state, drives phase transitions, validates and applies commands, and broadcasts updates. **`ClientGame`** submits commands and mirrors state. `GameManager` handles lifecycle/lobby/network/DI wiring.
- **Commands** (`IGameCommand`, split into Client/Server) are the *only* way state changes — they are serialized and sent over the transport (`Services/Transport`) via `CommandPublisher`. State changes propagate through observables + commands.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [anton-makarevich/MakaMek](https://github.com/anton-makarevich/MakaMek) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
