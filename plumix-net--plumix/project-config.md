---
trigger: always_on
description: This file defines expectations for coding agents working in this repository.
---

# AGENTS.md

This file defines expectations for coding agents working in this repository.

## Project Snapshot

- Platform: .NET 10
- UI stack: Avalonia
- Purpose: Flutter-like widget/rendering layer implemented in C#
- Main library: `src/Plumix`
- Example hosts: `src/Sample/*`
- Main solution: `src/Plumix.sln`

## Project Vision

- Build a Flutter-like framework in C# where `Widget`/`Element`/`RenderObject` concepts stay close to Flutter.
- Keep render object behavior and APIs close enough to Flutter to simplify rewriting controls from Dart to C#.
- Reuse Avalonia mainly as platform infrastructure: windowing/app host, lifecycle, input plumbing, and drawing backend abstractions.
- Keep layout/paint logic in the new framework, not in Avalonia control implementations (except thin host adapters).

## Expected End State (Definition of Done)

1. Applications are composed through Flutter-like widgets/state/lifecycle and rendered by framework-owned render objects.
2. Core rendering behavior lives in `src/Plumix/Rendering` and related framework layers, with minimal Avalonia-specific UI logic.
3. Desktop sample runs a widget app through `WidgetHost` (or an equivalent framework host), not only a render demo window.
4. Core primitives (box, flex, text, animation tick flow) are stable enough for straightforward Dart-to-C# control rewrites.
5. Project docs stay aligned with architecture boundaries and migration goals.

## Repository Map

- `src/Plumix`: core framework (`Foundation`, `Widgets`, `Rendering`, `UI`, scheduler/ticker pipeline).
- `src/Sample/Plumix.Sample`: shared sample app/widgets.
- `dart_sample`: reference sample app on real Flutter (Dart), kept in lockstep with `src/Sample/Plumix.Sample`.
- `src/Sample/Plumix.Desktop`: desktop entry point.
- `src/Sample/Plumix.Browser`: WebAssembly host.
- `src/Sample/Plumix.Android`: Android host.
- `src/Sample/Plumix.iOS`: iOS host.

## Progress Source of Truth

- Historical shipped changes: `CHANGELOG.md`
- Current status + global roadmap: `docs/FRAMEWORK_PLAN.md`
- Module entry points by task: `docs/ai/MODULE_INDEX.md`
- Non-negotiable behavior rules (architecture, package boundaries, versioning): `docs/ai/INVARIANTS.md`
- Mandatory Dart-to-C# porting workflow: `docs/ai/PORTING_MODE.md`
- Intentional divergences from Flutter: `docs/ai/DIVERGENCES.md`
- Sample parity tracker: `docs/ai/PARITY_MATRIX.md`
- Feature-to-tests map: `docs/ai/TEST_MATRIX.md`
- Iteration planning template: `docs/ai/FEATURE_TEMPLATE.md`
- Archived per-iteration notes (journal, not rules): `docs/ai/notes/`
- When task scope changes framework behavior, update tracking docs so agents can infer:
  - what is already done,
  - what remains,
  - what direction has priority now.
- `CHANGELOG.md` entries must be short (a few lines per change, no test-inventory prose). When the file exceeds roughly 100 KB, rotate the older half into `CHANGELOG-<year>-H<half>.md` and keep only the current period in `CHANGELOG.md`.

## Context Budget Protocol (For AI Agents)

1. Start with read order: `AGENTS.md` -> `docs/FRAMEWORK_PLAN.md` -> `docs/ai/MODULE_INDEX.md` -> targeted tests -> targeted implementation files.
2. Default scope for Dart-to-C# parity requests: close one control end-to-end in one request (`API/defaults/composition/states/layout/paint/tests`), not a sequence of micro-fixes.
3. Prefer entering unfamiliar subsystems through their tests (`docs/ai/TEST_MATRIX.md`); open implementation hotspot files (`Widgets/Scroll.cs`, `Rendering/Sliver.cs`, `Widgets/Navigation.cs`, `Widgets/Framework.Element.cs`, `SemanticsTreeTests.cs`) only when the task explicitly requires them.
4. Expand context proactively when needed to finish the current control in the same request; do not stop at partial parity unless blocked by a concrete missing primitive.
5. A task note (`docs/ai/FEATURE_TEMPLATE.md`, stored in `docs/ai/notes/`) is required only when an iteration ends blocked (unclosed parity with a concrete blocker) or introduces a divergence. Routine closed iterations need only `CHANGELOG.md` and matrix updates.
6. If sample behavior changes, update both `src/Sample/Plumix.Sample` and `dart_sample` in the same iteration and reflect status in `docs/ai/PARITY_MATRIX.md` (scope per `docs/ai/INVARIANTS.md` Sample Parity).
7. Before finishing, update docs with minimal deltas only (`CHANGELOG.md`, `docs/FRAMEWORK_PLAN.md`, and relevant `docs/ai/*` files) and keep `dotnet test src/Plumix.Tests/Plumix.Tests.csproj` green.

## Environment Requirements

- .NET SDK 10 preview (projects target `net10.0` and platform-specific TFMs).
- Avalonia tooling/workloads for browser/mobile targets where applicable.

## Local Reference Paths

- Flutter source: `/Users/egorozh/Documents/flutter/flutter`
- Avalonia source: `../Avalonia` (resolved: `/Users/egorozh/Flutter.Net.Local/Avalonia`)

## Common Commands

Run from repository root:

```bash
dotnet restore src/Plumix.sln
dotnet build src/Plumix.sln -c Debug
dotnet run --project src/Sample/Plumix.Desktop/Plumix.Desktop.csproj
dotnet run --project src/Sample/Plumix.Browser/Plumix.Browser.csproj
```

Platform-specific builds:

```bash
dotnet build src/Sample/Plumix.Android/Plumix.Android.csproj -c Debug
dotnet build src/Sample/Plumix.iOS/Plumix.iOS.csproj -c Debug
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Plumix-Net/Plumix](https://github.com/Plumix-Net/Plumix) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-14 -->
