---
trigger: always_on
description: This file gives an immediate, actionable orientation for an AI coding agent working in Parallax.
---

# Copilot instructions — Parallax Framework

This file gives an immediate, actionable orientation for an AI coding agent working in Parallax.

## 1) Big picture (where things live)
- **Entry points**: `gamemode/init.lua` (server) and `gamemode/cl_init.lua` (client). They include `framework/util.lua` and `framework/boot.lua`.
- **Framework auto-loads**: `framework/boot.lua` includes these directories in order: `libraries/`, `meta/`, `core/`, `hooks/`, `networking/`, `interface/`.
- **Global namespace**: Top-level framework table is `ax` (e.g. `ax.util`, `ax.config`, `ax.character`, `ax.item`, `ax.gui`). Add new subsystems under `framework/` or standalone modules under `modules/`.
- **Schema architecture**: Schemas extend the framework (see `minerva-hl2rp/gamemode/schema/` for example). Schemas have their own `boot.lua`, `factions/`, `classes/`, `items/`, `hooks/`, etc.

## 2) Naming / realm conventions (critical)
- **File prefixes determine realm**: `cl_` = client-only, `sv_` = server-only, `sh_` = shared. `ax.util:DetectFileRealm` and `ax.util:Include` follow this pattern automatically.
- **Prefer framework inclusion**: Place files under existing framework directories and rely on `ax.util:IncludeDirectory("...")` rather than manual `include`/`AddCSLuaFile` calls.
- **File naming**: Use lowercase with underscores (e.g. `chat_util.lua`, `sh_movement.lua`).

## 3) Loading & initialization patterns
- **Boot sequence**: `framework/boot.lua` auto-includes framework directories, then `GM:OnReloaded()` in `hooks/sh_hooks.lua` handles schema/module loading.
- **OnReloaded hook**: Calls `ax.faction:Include()`, `ax.class:Include()`, `ax.item:Include()`, `ax.module:Include()`, and `ax.schema:Initialize()` for hot-reloadable systems.
- **Module pattern**: Modules can have their own `OnReloaded()` method for initialization (see `modules/sh_ambients.lua` example).

## 4) Store & networking pattern (common & important)
- **Store factory**: `framework/store_factory.lua` provides the canonical store pattern. Stores have `registry`, `defaults`, `values`, and `networkedKeys`.
- **Config vs Options**: `ax.config` (server→client settings) vs `ax.option` (client→server preferences). Both use `ax.util:CreateStore()` with different `spec.name` and `authority`.
- **Networking setup**: Server registers nets automatically. `ax.config` broadcasts changes to clients via `spec.net.set`. `ax.option` syncs client preferences to server cache.
- **Adding networked settings**: Set `data.bNetworked = true` in store registration to enable automatic sync.
- **Persistence**: Stores auto-save to JSON files via `ax.util:ReadJSON`/`ax.util:WriteJSON`.

## 5) Debugging & developer workflow
- **No build step**: Drop gamemode folder into `garrysmod/gamemodes/`, start with `+gamemode your-schema-name`.
- **Developer mode**: Set `developer 1` ConVar to enable `ax.util:PrintDebug` output and detailed framework logging.
- **Linting**: Use `.glualint.json` configuration. Follow `STYLE.md` conventions (4 spaces, colon methods, K&R formatting).
- **Documentation**: LDoc generates docs to `public/` via `config.ld`. Use LDOC comments with `@realm`, `@param`, `@return`, `@usage`.
- **Version tracking**: `version.json` contains version/build info, updated by CI workflows.

## 6) Project-specific patterns and gotchas
- **Include helpers**: Prefer `ax.util:Include(path)` and `ax.util:IncludeDirectory(dir)` over raw `include`/`AddCSLuaFile` - they handle realm detection and path normalization.
- **Logging consistency**: Use `ax.util:Print*` helpers (Error, Warning, Debug, Success) instead of `MsgC`/`print` for framework-consistent output.
- **JSON persistence**: Framework provides `ax.util:ReadJSON` and `ax.util:WriteJSON` - stores automatically use these for config persistence.
- **LocalPlayer wrapper**: `boot.lua` wraps `LocalPlayer()` to return `ax.client` when set, respect this in client code.
- **Module hot-reload**: Modules can implement `MODULE:OnReloaded()` for initialization - see ambients module example.

## 7) Where to look (quick pointers)
- **Include/realm logic**: `gamemode/framework/util.lua`
- **Store/networking patterns**: `gamemode/framework/store_factory.lua`
- **Load order and initialization**: `gamemode/framework/boot.lua` + `gamemode/framework/hooks/sh_hooks.lua`
- **Entry points**: `gamemode/init.lua`, `gamemode/cl_init.lua`
- **Schema example**: `minerva-hl2rp/gamemode/schema/` (real working schema)
- **Style guide**: `STYLE.md`; **Overview**: `README.md`; **Module docs**: `MODULES.md`

## 8) Short actionable examples (copyable patterns)
- **Add client UI file**: Create `gamemode/framework/interface/cl_newpanel.lua` - auto-included, no manual AddCSLuaFile needed.
- **Register networked config**: `ax.config:Add("myKey", ax.type.string, "default", {bNetworked = true})` - auto-syncs to clients.
- **Create module with init**: Add `MODULE:OnReloaded()` method for hot-reload initialization (see modules/sh_ambients.lua pattern).

## 9) Styling & docs
- **Explicit style**: Repository defines strict rules in `STYLE.md`. Key points:
  - **Indentation**: 4 spaces (no tabs)
  - **Method notation**: Use colon (`:`) for methods expecting `self` (e.g. `function ax.util:Foo()`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Parallax-Framework/parallax](https://github.com/Parallax-Framework/parallax) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
