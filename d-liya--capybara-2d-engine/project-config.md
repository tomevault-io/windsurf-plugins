---
trigger: always_on
description: Shared guidance for coding agents working in this repository (Claude, Codex, Cursor, and others).
---

# AGENTS.md

Shared guidance for coding agents working in this repository (Claude, Codex, Cursor, and others).

## Project Overview

This is a **Capybara 2.5D game template** — a primitives-first API for fast iteration and AI-assisted feature building. The engine uses a component-based architecture with generated assets, scenes, archetypes, systems, and widgets.

The public engine interface is **`src/Game.ts`**. Prefer that facade over `src/core/`. Server features (save/load, auth, multiplayer) go through **`src/sdk/`**.

## Development Commands

```bash
# Development (watch mode with CSS and TypeScript bundling)
npm run dev

# Type checking
npm run typecheck
```

## Architecture & Code Organization

### Core Structure

- **`src/Game.ts`** — Public facade API (`createGame()`). This is the primary interface for all gameplay code.
- **`src/main.ts`** — Bootstrap entrypoint. Preloads assets/audio, creates loading gate, delegates to scene creation.
- **`src/core/`** — Runtime internals (camera, input, map, rendering, widgets manager). Do NOT import from here directly; use the GameAPI facade.
- **`src/scenes/`** — Scene entrypoints and orchestration. Each scene calls `createGame()` and orchestrates resources, archetypes, systems, inputs, and widgets.
- **`src/systems/`** — Per-frame gameplay logic (e.g., footstep audio, AI waves, combat). Systems receive `(dt, game)` and run each frame.
- **`src/archetypes/`** — Reusable entity defaults (body/render prefabs).
- **`src/widgets/`** — DOM HUD plugins mounted via `game.useWidget()`.
- **`src/data/`** — Generated JSON content and TypeScript handles. `assets.md` is the agent-facing manifest.
- **`src/sdk/`** — Capybara SDK facade for save/load, auth, multiplayer. Import from `src/sdk/index.ts`.

### Data Flow

1. **Generated assets** live in `src/data/` as JSON files with TypeScript exports
2. **Adapters** in `src/data/adapters.ts` convert flat JSON to engine shapes: `toMapData()`, `toArchetype()`, `toPlayerSprite()`
3. **Scenes** import generated handles and adapters, call `createGame()`, register archetypes/systems/widgets, spawn entities
4. **Systems** run per-frame logic via the GameAPI facade

## Key Architectural Rules

### Documentation Authority

This project uses **documentation-driven development**. When working with generated assets or engine patterns:

1. **`src/data/assets.md`** — Source of truth for generated maps, characters, props, widgets, audio, animation names, placement targets
2. **`src/scenes/SCENES.md`** — Scene composition facts (resources, archetypes, systems, inputs, widgets)
3. **`docs/recipes/`** — Optional implementation patterns (combat, inventory, NPCs, etc.)
4. **DO NOT** reverse-engineer `src/core/` or SDK internals — build from the docs and facades

### Coordinate System

- **Normalized coordinates**: 0-1000 per panel
- **Entity `x`, `y`**: Always **top-left** corner
- **Spawning methods**:
  - `spawnAtFeet(archetype, feetX, feetY, props)` — For characters (feetX = feet center, feetY = bottom edge)
  - `spawnCentered(archetype, centerX, centerY, props)` — For static props (arguments are center; entity stores top-left)
  - `placeProp(archetype, placement, props)` — For generated placement boxes (top-left + size)
- **Map extensions**: When stitching panels, world origin is the compiled map's top-left. Extending west/north shifts origin; adjust spawn coordinates to keep entities reachable.

### Asset Integration

When generating new assets (maps, characters, props, audio):

1. **Generation alone is incomplete** — assets must be wired into the game
2. After generation, update `src/data/assets.md` with new handles and facts
3. Import handles in scenes using `src/data/` adapters
4. For common assets (HUD, reference art, music, SFX), add to `src/data/common.json` as `{ name, url }`

### Player Entity Pattern

- Player is an entity, not a constructor argument
- Spawn player archetype in the scene, then call `game.setControlledEntity(playerId)`
- This keeps RPG and tower-defense style scenes unified

### Scene Creation Pattern

Scenes should:

- Return synchronously (no top-level `async`)
- Accept optional `onAudioReady` hook from loading gate for browser-gated playback (music, `AudioContext.resume()`)
- Also unlock looping BGM on first `keydown`/`pointerdown` — in local/dev `onContinue` is a no-op
- Register resources, archetypes, systems, inputs, widgets in scene setup
- Start SDK/save-load as async tasks that update resources when complete

Example:

```typescript
import { createGame, getAudio } from "../Game";
import { mapMain, toMapData, charPlayer, toArchetype } from "../data";

export function createMainScene({
  onAudioReady,
}: {
  onAudioReady?: (start: () => void) => void;
}) {
  const game = createGame({
    canvasId: "game",
    map: toMapData(mapMain),
    cameraEdgePadding: 120,
  });

  // Register resources, archetypes, systems, inputs, widgets
  game.defineArchetype("player", toArchetype(charPlayer, { speed: 190 }));
  const playerId = game.spawnAtFeet("player", 500, 820);
  game.setControlledEntity(playerId);

  // Browser-gated audio: production gate + first-input fallback (required in local/dev)
  let musicStarted = false;
  const startMusic = () => {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [d-liya/capybara_2d_engine](https://github.com/d-liya/capybara_2d_engine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-11 -->
