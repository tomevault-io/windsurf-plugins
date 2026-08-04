---
trigger: always_on
description: This document defines strict guardrails for any AI-assisted or automated agent contributions (including Copilot, custom prompt runners, or scripted refactors) working in the **Uno.Toolkit.UI** repository. Human contributors must also ensure generated changes comply before merge. It is the single source of truth for repo-wide orientation *and* the rules agents must follow; `CLAUDE.md` at the repo root is a thin pointer that includes this file.
---

# AI Agents Contribution & Coding Instructions

This document defines strict guardrails for any AI-assisted or automated agent contributions (including Copilot, custom prompt runners, or scripted refactors) working in the **Uno.Toolkit.UI** repository. Human contributors must also ensure generated changes comply before merge. It is the single source of truth for repo-wide orientation *and* the rules agents must follow; `CLAUDE.md` at the repo root is a thin pointer that includes this file.

<repository_orientation>

## Repository overview

Uno Toolkit ships higher-level UI controls for multi-platform Uno Platform / WinUI apps. Six NuGet packages are produced from this repo:

- `Uno.Toolkit.WinUI` — base controls (WinUI lineage). Built from `src/Uno.Toolkit.UI/Uno.Toolkit.WinUI.csproj`.
- `Uno.Toolkit.WinUI.Material` / `.Cupertino` — design-system style libraries on top of the base, under `src/library/`.
- `Uno.Toolkit.WinUI.Markup` / `.Material.Markup` — C# markup helpers, under `src/library/`.
- `Uno.Toolkit.WinUI.Simple` — minimal style set used by the `Simple` sample.

`src/Uno.Toolkit/Uno.Toolkit.csproj` is a tiny netstandard core (e.g. `ILoadable`) that all other projects reference. Despite the legacy `Uno.Toolkit.UI` folder name, the assembly is `Uno.Toolkit.WinUI` (see `AssemblyName` in the csproj).

## Solution layout

- `src/Uno.Toolkit.sln` — main solution. Includes libraries, samples, and runtime tests. Open this for full development.
- `samples/Uno.Toolkit.Samples.sln` — samples-only solution; references the libraries via project reference.
- `src/Uno.Toolkit.UI/` — all controls (`Controls/`), behaviors/extensions (`Behaviors/`), markup extensions (`Markup/`), helpers (`Helpers/`), themes (`Themes/`). XAML in `Controls/**/*.xaml` and `Behaviors/**/*.xaml` is merged into `Generated/mergedpages.xaml` at build time by `Uno.XamlMerge.Task` (config in `src/Uno.Toolkit.UI/xamlmerge-toolkit.props`). Don't hand-edit `Generated/`.
- `src/Uno.Toolkit.RuntimeTests/` — MSTest-based runtime tests; depends on `Uno.UI.RuntimeTests.Engine` and is hosted inside the sample apps.
- `samples/Uno.Toolkit.Samples/` — shared project (`.shproj`) with the sample UI; the platform heads in `samples/Uno.Toolkit.Samples.{Material,Cupertino,Simple}/` are the runnable apps.
- `ref/Uno.Themes` — sibling `Uno.Themes` repo content checked into this tree (not a git submodule despite appearances). Treat as vendored source unless told otherwise.

## Target frameworks and platform builds

Target frameworks are managed centrally:

- `src/tfms.props` defines `NetCurrent` (currently `net10.0`).
- `src/tfm-common-winui.props` expands library projects to `net9.0` + per-platform suffixes (`net9.0-ios`, `net9.0-android`, `net9.0-windows10.0.19041`, `net9.0-maccatalyst`); sample apps use `net10.0-*`.
- The Uno SDK version is pinned in `global.json` (`Uno.Sdk` and `Uno.Sdk.Private`).

The top-level `Directory.Build.props` also exposes `Build_Android`, `Build_iOS`, `Build_MacOS`, `Build_Windows` switches; non-Windows hosts default `Build_Windows=false`. The single-platform local-build flow (via `crosstargeting_override.props`) is documented in §4 below.

</repository_orientation>

<flow_orchestration>

### 1. Plan Node Default

- Enter plan mode for ANY non-trivial task (3+ steps or architectural decisions)
- If something goes sideways, STOP and re-plan immediately - don't keep pushing
- Use plan mode for verification steps, not just building
- Write detailed specs upfront to reduce ambiguity

### 2. Subagent Strategy

- Use subagents liberally to keep main context window clean
- Offload research, exploration, and parallel analysis to subagents
- For complex problems, throw more compute at it via subagents
- One tack per subagent for focused execution

### 3. Self-Improvement Loop

- After ANY correction from the user: update `specs/lessons.md` with the pattern (create the file/folder if it does not yet exist)
- Write rules for yourself that prevent the same mistake
- Ruthlessly iterate on these lessons until mistake rate drops
- Review lessons at session start

#### Where corrections are recorded

User corrections, "do this / never do that" rules, workflow guardrails, and tool-usage policies that should bind **every** agent working on this repo MUST be written to a checked-in, shared file:

- Repo-wide rules → `AGENTS.md` (this file).
- Skill-specific rules (e.g. how to use a particular tool/MCP) → the relevant `.claude/skills/<skill>/SKILL.md`.
- Domain lessons / postmortems → `specs/lessons.md`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [unoplatform/uno.toolkit.ui](https://github.com/unoplatform/uno.toolkit.ui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
