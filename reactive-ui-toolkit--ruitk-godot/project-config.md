---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A **React-style reactive UI library for Godot 4.x, written in plain GDScript** — the Godot leg of the
**Reactive UI Toolkit** family, sibling of the C#/Unity
[Reactive UI Toolkit — Unity](https://github.com/reactive-ui-toolkit/ruitk-unity). Function components
return a virtual tree; a fiber reconciler diffs each render and patches only what changed on the real
Godot `Control` tree. State lives in hooks.

The repo actually holds **four independently-versioned deliverables**, each with its own version and
release gate (see `.github/workflows/publish.yml`):

| Deliverable | Location | Language | Version source |
|---|---|---|---|
| The runtime addon | `addons/reactive_ui_toolkit/` | GDScript | `plugin.cfg` |
| The in-Godot-editor `.guitkx` plugin | `addons/reactive_ui_toolkit_editor/` | GDScript | `plugin.cfg` |
| VS Code + VS2022 `.guitkx` extensions | `ide-extensions/` | TypeScript / C# | `package.json` / `.vsixmanifest` |
| Docs site | `RuitkGodotDocs~/` | React + Vite | `package.json` |

`RuitkGodotDocs~` is named with a trailing `~` so the Godot importer skips it (it's a Node/Vite
project, not Godot content).

## Commands

### Runtime tests (headless GDScript — the primary test loop)

Godot has no compile step; "tests" are `tests/*.gd` scripts run under `--headless`, each `quit()`ing
non-zero on failure. Run them exactly like CI (`.github/workflows/test.yml`), **in this order**:

```bash
# 1. Build the class-name cache FIRST on a fresh clone: guitkx_build's two-pass parse gate
#    reload()s every generated .gd, whose global class_name references (V, Hooks, RuitkVNode, ...)
#    only resolve once .godot/global_script_class_cache.cfg exists (49/49 false parse fails without).
godot --headless --path . --editor --quit || true
# 2. Compile every examples/**/*.guitkx to its sibling .gd (the generated .gd is git-ignored)
godot --headless --path . --script res://tests/guitkx_build.gd
# 3. Re-scan so the just-generated .gd class_names register for the suites
godot --headless --path . --editor --quit || true
# 4. Run a suite (this is also how you run a SINGLE test file)
godot --headless --path . --script res://tests/core_test.gd
```

(On a working tree that already has a `.godot` cache, step 1 is a no-op and the old
build-then-scan habit still works — the strict order only matters on a fresh clone / CI.)

The suites: `core_test.gd` (reconciler/hooks/effects/bailout/context/keyed), `settings_test.gd`
(the `reactive_ui_toolkit/*` Project Settings bridge), `scheduler_test.gd` (the four-lane
`RuitkScheduler` — lanes/budgets/batching — plus sliced-render integration),
`strict_boundary_test.gd` (the cooperative `RuitkFail` error-boundary latch + strict mode), `style_test.gd`,
`router_match_test.gd` + `router_spine_test.gd`, `update_test.gd` (diff), `demos_test.gd` (renders
every demo — the real check that generated `.gd` render without error), `doom_game_test.gd` (the
Doom demo end-to-end), `guitkx_test.gd` (compiler + codegen + imports/resolver/codemod),
`hmr_test.gd` (Fast Refresh), `guitkx_editor_test.gd` + `guitkx_lsp_test.gd` (editor addon),
`contract_dump.gd -- --check` (GD↔TS grammar goldens). `tests/guitkx_migrate.gd` runs the 0.10.0
import codemod over `examples/` (idempotent — a clean tree reports 0 migrated). `bench*.gd` /
`microbench.gd` are benchmarks, not pass/fail tests.

### IDE tooling (TypeScript language server + VS Code extension)

```bash
cd ide-extensions/lsp-server && npm ci && npm run build && node --test out/test/*.test.js && node scripts/smoke.js
cd ide-extensions/vscode     && npm ci && npm run build          # F5 in VS Code to debug
```

`@vscode/vsce` and `ovsx` are invoked via `npx` (not deps) to keep `npm install` small. The bundled
language server embeds a native napi addon (`@gdscript-analyzer/core`), so a packaged `.vsix` is
**platform-specific**. See `ide-extensions/README.md` for packaging, the VS2022 build, and publishing.

### Docs site

```bash
cd RuitkGodotDocs~ && npm ci && npm run dev     # or: npm run build / npm run lint
```

## Architecture

### Runtime (`addons/reactive_ui_toolkit/core/`)

The library exposes global `class_name`s — **no autoload or plugin-enable is required to use the
runtime**; the classes are available as soon as the files exist. Enabling the plugin only adds the
`.guitkx` compile-on-save integration.

- **`v.gd` (`V`) / `vnode.gd` (`RuitkVNode`)** — the ~71 `V.*` factories and the immutable UI
  description. **Naming is 1:1 loyal to Godot (0.9.0, plans/archive/NAMING_LOYALTY_PROPOSAL.md +
  MIGRATION-0.9.md):** element factories are named exactly after the Godot class they create
  (`V.Button`, `V.VBoxContainer`); tags = official class names (any instantiable ClassDB Node
  class is a valid tag); events = `on` + PascalCase(signal) (`onPressed`); style keys = exact
  Godot property/theme/StyleBoxFlat names. Only structural factories are lowercase — `V.fc` is
  the function-component factory (GDScript reserves `func`, so it's not `V.func`).
- **`hooks.gd` (`Hooks`)** — the 23 hooks. Call only at the top of a render, in a stable order.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [reactive-ui-toolkit/ruitk-godot](https://github.com/reactive-ui-toolkit/ruitk-godot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-17 -->
