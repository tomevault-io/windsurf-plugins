---
trigger: always_on
description: Guide for AI agents and developers working on this codebase.
---

# AGENTS.md — Beyond Fable

Guide for AI agents and developers working on this codebase.

## Project summary

**Beyond Fable** is a browser-based, first-person procedural wilderness explorer built with **TypeScript**, **Vite**, and **Three.js**. Every page load generates a new seeded open world: terrain, forests, grass, rocks, water, sky, day/night, weather, and interactables. There is no backend, no downloaded asset packs, and no external game engine.

Design goals: Skyrim/Elden Ring-inspired scale and atmosphere, fully procedural geometry/textures/shaders, chunked streaming, and laptop-friendly performance via instancing, LOD, and fog.

---

## Commands

```bash
npm install          # install deps
npm run dev          # dev server (default http://localhost:5173)
npm run build        # tsc + production bundle → dist/
npm run preview      # serve production build
npm run typecheck    # TypeScript only
```

Always run `npm run typecheck` (or `npm run build`) after substantive edits.

---

## Tech stack

| Layer    | Choice                                                                  |
| -------- | ----------------------------------------------------------------------- |
| Language | TypeScript (strict)                                                     |
| Bundler  | Vite 5                                                                  |
| 3D       | Three.js 0.165 (WebGL2)                                                 |
| Assets   | 100% procedural (canvas textures, shaders, generated geometry)          |
| Physics  | Custom: analytic terrain height + cylinder push-out (no physics engine) |

---

## Directory layout

```
beyond-fable/
├── index.html              # HUD DOM + canvas mount (#app)
├── src/
│   ├── main.ts             # Entry: seed from URL, start Game
│   ├── config.ts           # Typed exports backed by settings/global-defaults.json
│   ├── settings/
│   │   └── global-defaults.json # ★ Editable global world/gameplay/render defaults
│   ├── styles.css          # HUD / overlay styles
│   │
│   ├── core/               # Application shell
│   │   ├── Game.ts         # Main loop, quality fallback, wires all systems
│   │   ├── Renderer.ts     # WebGLRenderer + EffectComposer post chain
│   │   ├── CameraController.ts  # FPS walk + fly mode, terrain collision
│   │   └── Hud.ts          # FPS, seed, time/weather, interact UI, hack menu
│   │
│   ├── render/
│   │   └── AerialFogPass.ts # Scene render + depth-based distance-fog composite
│   │
│   ├── world/              # Scene content & simulation
│   │   ├── World.ts        # ★ Top-level world composition root
│   │   ├── Terrain.ts      # Analytic height field + chunk mesh builder
│   │   ├── Biomes.ts       # Moisture, forest mask, ground color, placement rules
│   │   ├── ChunkManager.ts # ★ Chunk streaming, LOD, build/grass/tree-batch queues
│   │   ├── FarTerrain.ts   # Distant ~11 km horizon mesh (1 draw call)
│   │   ├── Environment.ts  # ★ Day/night, weather, wind, lights, rain, torch
│   │   ├── Sky.ts          # Sky dome mesh + shader uniforms
│   │   ├── Water.ts        # Water chunk meshes + player ripple state
│   │   ├── Vegetation.ts   # Chunk-streaming facade over the vegetation/ grammar
│   │   ├── GrassSettings.ts # Runtime-tunable grass params (hack menu)
│   │   ├── Rocks.ts        # Props class: boulders, stones, logs, shrubs, flowers
│   │   ├── Structures.ts   # Fantasy landmarks (spires, floating islands, fossils…)
│   │   ├── Glow.ts         # Bioluminescent plants + fireflies (night)
│   │   ├── SnowTrail.ts    # Temporary footstep depressions in snow
│   │   ├── Clearings.ts    # Deterministic campfire-clearing test (shared by veg + POIs)
│   │   ├── Fire.ts         # Lightable campfire: procedural flame/ember shaders
│   │   └── Interactables.ts # POIs + InteractionSystem (E key)
│   │
│   ├── vegetation/         # ★ Procedural flora kit (no external assets)
│   │   ├── Botany.ts       # Profile/tier/canopy/skeleton type definitions
│   │   ├── TreeCatalog.ts  # TREE_CATALOG presets (spruce, pine, beech, birch, karst, snag, oak, cherry)
│   │   ├── Branching.ts    # Recursive branch growth (tropisms, phyllotaxis, crown envelope)
│   │   ├── MeshForge.ts    # Append-only geometry accumulator (shared by all builders)
│   │   ├── Limbs.ts        # Tube hierarchy meshing for bark
│   │   ├── Foliage.ts      # Real leaf / needle-fan geometry builders
│   │   ├── CardAtlas.ts    # Bakes leafy twigs to a per-species atlas; scatters cluster cards
│   │   ├── Assemble.ts     # Profile + SeedStream → bark + foliage geometry (LOD-aware)
│   │   ├── Undergrowth.ts  # Shrubs, ferns, flowers (same grammar, bush-tuned)
│   │   ├── Sward.ts        # Grass blade-tuft geometry
│   │   └── SeedStream.ts   # Label-forkable seeded RNG for the flora kit
│   │
│   ├── procedural/
│   │   ├── Noise.ts        # Seeded 2D simplex + fBm + ridged
│   │   ├── Textures.ts     # CanvasTexture generators (bark, rock, ground)
│   │   └── Materials.ts    # MaterialLibrary (shared PBR materials per world)
│   │
│   ├── shaders/            # GLSL source strings (imported as TS modules)
│   │   ├── noiseGLSL.ts    # Shared hash/fBm for sky & water
│   │   ├── sky.ts
│   │   ├── grass.ts

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xikhar/beyond-fable](https://github.com/xikhar/beyond-fable) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
