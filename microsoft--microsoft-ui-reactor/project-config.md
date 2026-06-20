---
trigger: always_on
description: >
---


# Reactor — Getting Started

> **Prefer the plugin.** This file is preserved for environments that don't
> support the Copilot CLI / Claude plugin loading model. If you have a plugin
> SDK available, install / load the `reactor` plugin (under
> `plugins/reactor/` in source, or `agentkit/plugins/reactor/` in the NuGet).
> The plugin splits this content into focused per-skill files and is materially
> cheaper to load than this monolith.



Reactor is a **React-inspired functional projection for WinUI 3**. You write
functions that return lightweight element descriptions; a reconciler diffs
old vs new trees and patches real WinUI controls. State changes trigger
re-renders automatically. No XAML. No data binding. No ViewModels.

## Which mode are you in? (read this first)

Reactor ships as a NuGet package — apps reference it as
`<PackageReference Include="Microsoft.UI.Reactor" Version="…" />` (or
`#:package Microsoft.UI.Reactor@…` for single-file). The package carries
the framework, the analyzers, and an **agent kit** (signatures index +
this SKILL.md). Two paths:

| Mode | How to detect | Bootstrap |
|---|---|---|
| **Selfhost** — you're in a Reactor source clone (`src/Reactor/Reactor.csproj` exists) | The repo's `local-nupkgs/` folder is the package source — see `nuget.config` at repo root. | Build `mur` once, then **`mur pack-local`** to populate `local-nupkgs/Microsoft.UI.Reactor.0.0.0-local.nupkg`. Re-run after framework changes. |
| **Consumer** — you're in an app that depends on Microsoft.UI.Reactor | No `src/Reactor/` next to your project. | Nothing extra — the package already carries the analyzers and agent kit. If `mur` is on PATH, `mur --skill` and `mur --api` print the embedded docs. Otherwise read `<package-cache>/microsoft.ui.reactor/<version>/agentkit/`. |

If you're in selfhost and `local-nupkgs/` is empty, restore will fail with
"package Microsoft.UI.Reactor 0.0.0-local was not found." Run `mur pack-local`
to fix it.

### Bootstrap (selfhost, fresh clone)

```powershell
# Build the CLI; on first build the SignaturesGen project also writes
# skills/reactor.api.txt as part of its AfterBuild target.
dotnet build src/Reactor.Cli -p:Platform=ARM64

# `mur` mirrors itself to <repo>/bin/<arch>/. Add that to PATH or invoke directly.
.\bin\arm64\mur.exe pack-local
```

After this, any project under the clone resolves
`Microsoft.UI.Reactor 0.0.0-local` from `local-nupkgs/` automatically (the
repo-level `nuget.config` configures it). A consumer **outside** the clone
needs a project-local `nuget.config` pointing at the absolute path of
`<repo>/local-nupkgs/`.

## Where to find docs (`mur --skill`, `mur --api`)

The `mur` CLI ships these embedded — works from any directory:

| Command | What it prints | Source |
|---|---|---|
| `mur --skill` | This SKILL.md | embedded in `mur` |
| `mur --api`   | The signatures index (≈12K tokens, every factory/modifier/hook/Theme token/enum) | embedded in `mur` |
| `mur --regen-api` | Rebuilds `skills/reactor.api.txt` from a freshly-built `Reactor.dll` (selfhost only) | rebuilds `tools/Reactor.SignaturesGen` |
| `mur check <path>` | **Is** the build (same exit code as `dotnet build`); adds one-line diagnostics with skill pointers for known REACTOR_* IDs and `→ try:` did-you-mean suggestions | wraps MSBuild |

A consumer who doesn't have `mur` can read the same files directly from the
NuGet cache:

```
%USERPROFILE%\.nuget\packages\microsoft.ui.reactor\<version>\agentkit\
├─ SKILL.md                  ← this file
├─ reactor.api.txt           ← signatures index
└─ skills\
   ├─ async.md, design.md, commanding.md, navigation.md, forms.md,
   │  input.md, charts.md, dsl-reference.md, devtools.md, perf-tips.md
   └─ recipes\
      ├─ index.md            ← intent → recipe map
      └─ <name>.cs           ← paste-ready single-file programs
```

When SKILL.md or a recipe references `skills/foo.md`, a consumer agent
reads it from `agentkit/skills/foo.md` in the package cache. Selfhost
agents read it from `<repo>/skills/foo.md`.

## API signatures index — load this before grepping source

[`skills/reactor.api.txt`](skills/reactor.api.txt) is a generated, alphabetized
flat list of every public Factory, Modifier, Hook, Theme token (with WinUI
resource key), and enum in Reactor. **Load this when you need to confirm a
signature.** It replaces grepping `src/Reactor/Elements/*.cs` and walking the
sub-skills' tables.

- **Local / selfhost:** the file is committed at `skills/reactor.api.txt`.
  Run `mur --api` to print it. Run `mur --regen-api` after framework changes.
- **NuGet consumer:** the same file ships in the package at
  `<package-cache>/microsoft.ui.reactor/<version>/agentkit/reactor.api.txt`
  (typically `%USERPROFILE%\.nuget\packages\microsoft.ui.reactor\<version>\agentkit\reactor.api.txt`).
  If `mur` is on PATH, `mur --api` prints the embedded copy.

## Recipes — paste-ready snippets indexed by intent

[`skills/recipes/`](skills/recipes/) holds compilable single-file recipes for
the most common Reactor patterns. **Load a recipe instead of synthesizing
from skill prose.** See [`skills/recipes/index.md`](skills/recipes/index.md)
for the intent → recipe map. Available today: list-add-delete, sidebar-nav,
form-with-validation, async-fetch-list, themed-card, canvas-positioning,
named-styles, calendar-multiselect.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [microsoft/microsoft-ui-reactor](https://github.com/microsoft/microsoft-ui-reactor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
