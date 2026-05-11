---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

This is the **draw-steel-codex** repository — the Lua mod source code for [DMHub](https://dmhub.app), a tabletop RPG virtual tabletop (VTT). Specifically, it implements the **Draw Steel** (MCDM) game system on top of the DMHub engine. The code runs inside the DMHub app; there is no standalone build process, test runner, or linter to invoke from the command line.

## How the Code is Loaded

`main.lua` at the root is the module entry point. It contains a flat list of `require(...)` calls that load every file in the project. Each `require` uses the pattern `ModuleName_XXXX.FileName`, where `ModuleName_XXXX` is a subdirectory name (with a hex suffix that acts as a module ID). Files are loaded in order — dependencies must come before the files that use them.

Each Lua file begins with:
```lua
local mod = dmhub.GetModLoading()
```
This gives access to the current module interface. The `mod` object is used to track module lifecycle (e.g., `mod.unloaded`).

**IMPORTANT: Do not create new Lua files.** Lua files are registered through the DMHub module system and will not auto-load just by being placed on disk. Adding a `require` in `main.lua` for a file that hasn't been registered will cause a load failure. If new code is needed, add it to an existing file in the appropriate module. If a new file is truly necessary, ask the user to create and register it through the DMHub module system.

## Repository Structure

Each top-level directory is a "mod" (module) loaded by DMHub. Key layers:

| Directory | Purpose |
|---|---|
| `Definitions/` | LuaLS type stubs for the DMHub engine API (not executed — documentation only). All engine globals (`dmhub`, `gui`, `game`, `creature`, etc.) are declared here. |
| `DMHub Utils/` | Shared utility library: `Utils.lua` (table/string helpers), `GoblinScript.lua` (formula expression evaluator), `CoroutineUtils.lua`, `MarkdownRenderUtils.lua`. |
| `DMHub Core UI/` | Core UI framework: `Gui.lua` wraps the engine `gui` global, `Hud.lua`, `DockablePanel.lua`, `Scrollable.lua`, etc. |
| `DMHub Core Panels/` | Application panels: Chat, Character panel, Map tools, Compendium, Dev tools, etc. |
| `DMHub Game Rules/` | Base game rules system: `BasicRules.lua`, `ActivatedAbility.lua`, `Creature.lua`, `Character.lua`, `Class.lua`, `Condition.lua`, `Equipment.lua`, etc. This layer is generic/system-agnostic. |
| `Draw Steel Core Rules/` | The Draw Steel (MCDM) game system implementation built on top of DMHub Game Rules. `MCDMRules.lua` calls `GameSystem.ClearRules()` then sets DS-specific names (Stamina, Characteristics, Power Rolls, etc.). Most `MCDM*.lua` and `DS*.lua` files here extend or override base game types. |
| `Draw Steel Character Builder/` | New character creation wizard UI and state machine. |
| `Draw Steel UI/` | DS-specific UI panels (action bar, character sheet, class/kit editors, initiative, etc.). |
| `Draw Steel V/` | Newer DS feature panels (encounter, heroes, negotiation, downtime, fishing, chessboard, etc.). |
| `Draw Steel Ability Behaviors/` | Individual ability behavior implementations (`AbilityDamage`, `AbilityForcedMovementLoc`, `AbilityTemporaryEffects`, etc.). |
| `Draw Steel Modifiers/` | Modifier implementations (`ModifierCaptain`, `ModifierForcedMovement`, `ModifierInvisibility`, etc.). |
| `Downtime Projects/` | Downtime project system (rules + UI). |
| `DMHub Compendium/` | Compendium browser and editors for game content. |
| `DMHub CharacterSheet Base/` | Base character sheet framework. |
| `DocumentSystem/` | Rich document/journal system with Markdown, images, embedded dice rolls, etc. |

## Core Architecture Patterns

### Game Types
Game objects are declared with `RegisterGameType("TypeName")` or `RegisterGameType("TypeName", "ParentType")`. This registers a type in the engine's serialization system. You then add default fields and methods directly on the global:
```lua
CharacterCondition = RegisterGameType("CharacterCondition", "CharacterFeature")
CharacterCondition.name = "New Condition"
CharacterCondition.tableName = "charConditions"
function CharacterCondition:SoundEvent() ... end
```

Fields prefixed with `_tmp_` are **transient** -- the engine skips them during serialization. Use `_tmp_` fields for ephemeral runtime state that should not be saved to the database or sent over the network. Reading a `_tmp_` field that was never set will error; use `obj:try_get("_tmp_foo")` for safe access.

Extending a type from another file (common in `Draw Steel Core Rules/`):
```lua
-- Extend creature with Draw Steel fields
creature.minion = false
local g_base = creature.Invalidate
function creature:Invalidate()
    g_base(self)
    -- DS-specific invalidation
end
```

### Data Tables
Game data is stored in named tables accessed via `dmhub.GetTable("tableName")`. Iterate with `unhidden_pairs(t)` (skips soft-deleted entries). Write with `dmhub.SetAndUploadObject(tableName, id, obj)`.

### Modifying Token Properties

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [VerisimLLC/draw-steel-codex](https://github.com/VerisimLLC/draw-steel-codex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
