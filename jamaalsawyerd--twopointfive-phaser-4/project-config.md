---
trigger: always_on
description: This repository contains a Phaser 4 demo/game plus a reusable 2.5D engine/plugin called TwoPointFive. The core engine lives in `src/twopointfive/`, while the Phaser game/demo glue lives in `src/phaser-game.ts` and `src/game/`.
---

# AGENTS.md

## Repository overview

This repository contains a Phaser 4 demo/game plus a reusable 2.5D engine/plugin called TwoPointFive. The core engine lives in `src/twopointfive/`, while the Phaser game/demo glue lives in `src/phaser-game.ts` and `src/game/`.

There is also an older ImpactJS demo/port in `impact-version/`, served separately from `impact-index.html`.

## Essential commands

Use the Node version in `.nvmrc` (`24.11.1`).

```bash
npm install
npm run build
npm start
npm run lint
npm run lint:fix
npm run format
npm run format:check
```

Notes:
- `npm run build` executes `node build.js` and writes the bundled demo to `dist/game.js`.
- `npm start` rebuilds first, then runs `dev-server.js` on port `8080`.
- `npm run lint` only targets `src/`.
- `npm run format` runs ESLint fixers on `src/` and Prettier on `src/**/*.ts`.

## Project structure

- `src/twopointfive/` — engine/plugin code: renderer, cameras, world maps, collision, entity base class, timer, utilities, and public exports.
- `src/game/` — Phaser demo game objects and gameplay entities.
- `src/phaser-game.ts` — Phaser scene bootstrap, asset loading, plugin setup, HUD, input, and spawn logic.
- `media/` — shared assets used by the demo.
- `impact-version/` — original ImpactJS demo, engine copy, and Weltmeister editor files.
- `index.html` — Phaser demo entry point.
- `impact-index.html` — Impact demo entry point.
- `weltmeister.html` — Weltmeister editor entry point.
- `dev-server.js` — local static server plus Weltmeister browse/glob/save API replacements.
- `build.js` — esbuild bundle script.

## Architecture and control flow

### Phaser path

1. `src/phaser-game.ts` registers `TwoPointFivePlugin` as a global Phaser plugin.
2. The scene uses `scene.tpf` (scene plugin) to create the engine renderer, camera, and game state.
3. `MainScene.preload()` loads textures, audio, JSON level data, and web fonts.
4. `MainScene.create()` wires tilesets, light-map pixels, entity classes, HUD objects, pointer lock, and the `Extern` object that draws the 2.5D world inside the Phaser scene.
5. `MainScene.update()` forwards delta time to `tpf.update(delta)` and handles spawn timers.
6. `GameState.loadLevel()` builds maps, collision, lighting, culled sectors, and entities from Impact-style level JSON.
7. Entities update themselves through the shared `EntityContext` and render through the engine renderer.

### Engine path

- `src/twopointfive/entity.ts` is the base physics/rendering entity. It handles velocity, gravity, collision trace, animation updates, and light/sector updates.
- `src/twopointfive/game.ts` owns the level, entity registry, collision map, light map, and pairwise entity collision checks.
- `src/twopointfive/world/map.ts` and `wall-map.ts` build tile meshes from level layers.
- `src/twopointfive/world/light-map.ts` converts light-layer data plus image pixels into per-tile colors.
- `src/twopointfive/renderer/renderer.ts` batches quads into WebGL and handles fog, camera, and texture uploads.
- `src/twopointfive/two-point-five-plugin.ts` bridges Phaser with the engine and exposes `scene.tpf`.
- `src/twopointfive/render-adapter.ts` is a seam for the world render path (`LegacyWebGLRenderAdapter` is the only implementation).
- `src/twopointfive/entity-display-adapter.ts` is a seam for entity rendering. The only implementation is `LegacyEntityDisplayAdapter` (a no-op): entities draw themselves as WebGL billboard quads inside the Extern pass, so they depth-test against walls and pick up fog and lighting. A `ProjectedSpriteEntityDisplayAdapter` (Phaser `Image` sprites) was tried but retired — see the Phaser 4 rendering constraints below.

## Phaser 4 rendering constraints (important — verified against phaser@4.2.0)

Before proposing any "move the renderer to Phaser-native" work, know these hard facts about Phaser 4:

- **No 3D `Mesh`/`Plane` GameObject.** Phaser 4 removed both (see `node_modules/phaser/changelog/v4/4.0/MIGRATION-GUIDE.md`: *"`Mesh` and `Plane` have been removed… proper 3D support is planned for the future."*). Only `Mesh2D` exists, which is strictly 2D (`[x,y,u,v]` vertices, no z, no per-vertex color, no projection). **There is nothing to port the 2.5D world geometry onto.** Do not plan a "world → Phaser Mesh" migration; it is not possible in 4.x.
- **GameObjects do not use the WebGL depth buffer.** `setDepth()` is purely a display-list sort key (painter's algorithm), not GPU depth testing. Two consequences:
  - True per-pixel occlusion only happens inside the custom WebGL pass (the `Extern`). The custom renderer here is the *sanctioned* Phaser 4 way to do 3D, not legacy debt.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jamaalsawyerd/twopointfive-phaser-4](https://github.com/jamaalsawyerd/twopointfive-phaser-4) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-25 -->
