---
trigger: always_on
description: > This file exists so Claude (or any LLM) can resume work on this codebase without prior conversation history.
---

# CLAUDE.md — Context for AI Assistants

> This file exists so Claude (or any LLM) can resume work on this codebase without prior conversation history.

## Project Identity

**Uncivilised** (base game) — a browser-based 4X strategy game. This is the **open-source** repo containing the complete game engine. The AI diplomacy system lives in a separate private repo (`uncivilised-diplomacy`) and plugs in at build time.

**Live URL:** [uncivilized.fun](https://uncivilized.fun)
**Created:** March 2026
**Game version:** 5 (`GAME_VERSION = 5` in both `src/constants.js` and `server.py`)

## Architecture: Plugin Split

The codebase was split into two repos:

- **This repo** (`uncivilised-game-base`) — open source, contains the full game engine
- **Private repo** (`uncivilised-diplomacy`) — AI diplomacy, game mods, AI faction logic

### How the Plugin Works

1. `src/diplomacy-api.js` defines the plugin interface with no-op stubs for all diplomacy functions
2. All modules import diplomacy functions from `diplomacy-api.js` (never directly from diplomacy files)
3. `esbuild.config.mjs` checks for `../uncivilised-diplomacy/src/plugin.js` at build time
4. If found: generates `src/_diplomacy-plugin.gen.js` that imports the plugin and calls `init(registerDiplomacyPlugin)`
5. If not found: generates a no-op loader — game works without diplomacy

### Plugin Interface (`src/diplomacy-api.js`)

Exports these wrapper functions that delegate to the plugin:

**From diplomacy.js:** `getRelationLabel`, `establishTradeRoute`, `cancelTradeRoute`, `renderDiplomacyPanel`, `renderDiplomacyList`, `renderRankingsView`, `openChat`, `renderDiplomacyActions`, `renderChatMarkdown`, `updateDiploActions`, `appendChatMessage`, `appendChatAction`, `sendChatMessage`, `showDiplomacyProposal`, `processCharacterAction`

**From game-mods.js:** `applyGameMod`, `showModBanner`, `getModCombatBonus`, `getModYieldBonus`

**From ai.js:** `processAITurns`, `processBarbarianTurns`, `processAICommitments`, `moveAIUnitToward`

### Import Convention

The diplomacy repo's files use `@game/` prefixed imports to reference base-game modules:
```js
import { FACTIONS } from '@game/constants.js';
import { game } from '@game/state.js';
```
esbuild resolves these via the `alias` config in `esbuild.config.mjs`.

## Frontend Architecture

ES modules under `src/`, bundled by esbuild into `game.js` (IIFE format, gitignored). Key modules:

| Module | Purpose |
|--------|---------|
| `src/main.js` | Entry point, wires modules, exposes window globals |
| `src/state.js` | Shared mutable state (game, canvas, camera, drag) |
| `src/constants.js` | Game data: terrain, units, buildings, techs, factions |
| `src/render.js` | Canvas2D hex rendering, visibility, minimap |
| `src/terrain-render.js` | Painterly terrain detail (noise, splotches, trees) |
| `src/hex.js` | Hex grid utilities, coordinate math, neighbor lookups |
| `src/map.js` | Map generation (tectonic plates -> biomes -> rivers) |
| `src/assets.js` | Terrain tile preloading, portrait/icon asset management |
| `src/input.js` | Mouse/touch/keyboard handlers, camera, zoom |
| `src/ui-panels.js` | Build, research, civics, victory, selection panels |
| `src/units.js` | Unit creation, selection, movement, pathfinding |
| `src/combat.js` | Combat resolution, promotions, city attacks |
| `src/buildings.js` | Building mechanics, great persons, pantheon |
| `src/improvements.js` | Worker actions, tile improvements, settlers |
| `src/turn.js` | End-of-turn processing, income, maintenance |
| `src/housing.js` | City housing and population mechanics |
| `src/minor-factions.js` | Barbarian and minor faction system |
| `src/discovery.js` | Fog of war, faction discovery, first contact |
| `src/events.js` | Event log, toasts, notifications, rumours |
| `src/save-load.js` | Save/load (localStorage + API fallback) |
| `src/leaderboard.js` | Leaderboard UI and player ranking display |
| `src/rankings.js` | Rankings and stats calculation |
| `src/feedback.js` | In-game feedback UI |
| `src/resource-icons.js` | Resource icon and display utilities |
| `src/utils.js` | General utility functions |
| `src/diplomacy-api.js` | Plugin interface — stubs + registerDiplomacyPlugin() |
| `src/ai-diplomacy.js` | AI-to-AI diplomatic system |

**Build:**
- `npm run build` — one-shot build
- `npm run watch` — rebuild on file changes
- `npm run dev` — watch + serve via `server.py`

**Testing:**
- `npm test` — run the full test suite (Vitest, ~170 tests)
- `npm run test:watch` — re-run tests on file changes
- **Always run `npm test` before committing, pushing, or creating a PR.** Tests must pass. If tests fail, fix the issue before proceeding.
- Tests live in `tests/` with fixtures in `tests/fixtures.js` and browser global stubs in `tests/setup.js`
- Test naming: use `test()` (not `it()`), add `()` after function names in `describe()`, use dummy faction names (`faction_a`, `faction_b`) not real ones
- When fixing bugs or changing game logic, add a test that covers the fix when practical. Pure UI changes, render tweaks, and trivial one-liners don't need tests.

## Backend (Python FastAPI)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [uncivilised-game/uncivilised-game-base](https://github.com/uncivilised-game/uncivilised-game-base) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
