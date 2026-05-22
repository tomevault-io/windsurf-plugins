---
trigger: always_on
description: Concise working notes for AI agents contributing to this Minecraft Education add-on. Focus on patterns actually used in the repo.
---

## Educator Tools – AI Coding Guide

Concise working notes for AI agents contributing to this Minecraft Education add-on. Focus on patterns actually used in the repo.

### Core Purpose

Modular Minecraft Education Add-On providing classroom management (teams, assignments, focus mode, timers, world settings, etc.). Free and open-source (LGPL v3).

### Build & Profiles (Regolith)

Regolith drives generation of behavior/resource packs via filter profiles in `regolith/config.json`.
Common profiles:

- `educator_tools` – main add-on (education build target, whitelists `educator_tools`, `letter_blocks`, `scripting_setup`, `addon_setup/educator_tools`).
- `more_letter_blocks` – optional companion add-on (whitelists `addon_setup/more_letter_blocks`, `more_letter_blocks`).
- `debug` – adds `debug_say_function_name` filter on top of the default profile.
  Run builds via VS Code tasks or manually:

```powershell
cd regolith
regolith run            # default profile
regolith run educator_tools
regolith run more_letter_blocks
regolith run debug      # debug profile
```

### Scripts & Tooling

Install dependencies:

```powershell
\.scripts\install_dependencies.ps1  # npm install + regolith install-all
```

Build / Debug tasks call `\.scripts\build.ps1` / `debug.ps1` (adds Loopback exemptions for debugger).

### TypeScript & Bundling (ModularMC)

The project uses the **ModularMC** Regolith filter (successor to System Template). Full docs: <https://modular-mc-docs.readthedocs.io/en/stable/>.

ModularMC is a single filter (`modular_mc`) that handles both file mapping and script compilation:

- **Module folder**: any directory under `regolith/filters_data/modular_mc/` containing a `_map.ts` file becomes a module.
- **`_map.ts`**: TypeScript file exporting `MAP` (array of `{ source, target, … }` entries that map module files to BP/RP output paths) and/or `SCRIPTS` (array of TS/JS paths to compile with Esbuild).
- **Auto-mapping**: common file-suffix-to-output-path rules live in `regolith/filters_data/modular_mc/auto-map.ts` (e.g. `.block.json` → `BP/blocks`, `.behavior.json` → `BP/entities`).
- **Scope**: variables for JSON/text templates are passed via the `scope` property on each MAP entry (or imported from `regolith/filters_data/scope.json`). No implicit `_scope.json` merging – everything is explicit in `_map.ts`.
- **Esbuild**: configured per-profile in `config.json` under `esbuild.settings` (`outfile`, `sourcemap`, `minify`, `external`). Output goes to `BP/scripts/edu_tools/main.js`. Dependencies are managed through `regolith/deno.json` (Deno-style import map).
- **Script entry for Edu Tools**: `regolith/filters_data/modular_mc/educator_tools/logic/main.ts` → initializes `ModuleManager` on `world.afterEvents.worldLoad`. Scripts in `logic/subscripts/` are auto-collected via `getScriptFiles()` in the module's `_map.ts`.

### Module Pattern

`ModuleManager` (singleton) wires services from `logic/subscripts/modules/*/*.service.ts` implementing `Module` interface (`id`, optional `initialize`, `registerScenes`, `getMainButton`). Scene-centric navigation handled by `SceneManager` (singleton) with dynamic scene registry and history.
Add UI button: implement `getMainButton()` returning `{ labelKey, iconPath, handler, weight }` then let `ModuleManager` auto-register.

### Scenes & Navigation

Scenes registered via `sceneManager.registerScene(<name>, factory)`. Use a `SceneContext` to maintain history; back navigation uses `goBack()` or `goBackToScene()`. Script events consumed through `edu_tools:scene_manager` channel.

### Storage & Persistence

Per-module state stored with `@shapescape/storage` (`PropertyStorage`). Acquire sub-storage via `getSubStorage("<module>")`. For dynamic teams: `TeamsService` synthesizes system teams (teachers/students/all/player) and persists editable ones. Avoid direct world queries where cached storage exists.

### Adding a New Feature Module (Example)

1. Create folder `logic/subscripts/modules/<feature>/` (inside `modular_mc/educator_tools/`).
2. Implement `<feature>.service.ts` exporting `id`, optional `initialize`, `registerScenes`, and `getMainButton`.
3. Register any scenes in `registerScenes(sceneManager)`.
4. Use `ModuleManager` constructor sequence (automatic) – do NOT manually instantiate `ModuleManager`.
5. The module's `_map.ts` auto-collects all `.ts` files under `subscripts/` – no manual SCRIPTS registration needed.

### Adding a New ModularMC Module

1. Create a folder under `regolith/filters_data/modular_mc/<module_name>/`.
2. Add a `_map.ts` exporting `MAP` and/or `SCRIPTS`.
3. Use auto-mapping suffixes (see `auto-map.ts`) or explicit `{ source, target }` entries.
4. If scripts are needed, list them in `SCRIPTS` or use a helper like `getScriptFiles()`.
5. Whitelist the new module path in the appropriate profile in `regolith/config.json`.

### Manifest & Dependencies


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ShapescapeMC/Educator-Tools](https://github.com/ShapescapeMC/Educator-Tools) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
