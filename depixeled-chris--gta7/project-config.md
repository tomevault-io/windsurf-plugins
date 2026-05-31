---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A browser-based, GTA-style 3D open-world **vertical slice** built from scratch in TypeScript + Three.js: a procedural night city you can drive a car around, get out, and walk on foot, with ambient traffic and pedestrians. It is a tech demo, not a full game. (The name is a joke.)

## Commands

```bash
npm install                 # first-time setup
npm run dev                 # Vite dev server (hot reload) — play at the printed URL
npm run build               # tsc --noEmit (typecheck) THEN vite build -> dist/
npm test                    # Vitest: pure-logic unit tests (node env)
npm run test:watch          # Vitest in watch mode
npm run smoke               # build + headless-Chromium render check (see below)
npm run test:e2e            # build + render + gameplay (keyboard) + touch/mobile tests
npm run preview             # serve the built dist/
```

Run a single test file or case:

```bash
npx vitest run src/vehicles/VehicleModel.test.ts
npx vitest run -t "caps speed at maxSpeed"
```

The browser tests (`scripts/smoke.mjs`, `scripts/interaction.mjs`) require Chromium once: `npx playwright install chromium`. Both self-host the built app. `smoke` fails on any console/page error and decodes a screenshot to assert the scene actually rasterized (color diversity + lit-pixel fraction) rather than rendering a blank canvas. `interaction` drives the real game (keyboard) to assert gameplay: building collision, carjacking, shoving, pedestrian braking, and WASTED. `touch` runs a mobile (touch) context to assert the on-screen joystick drives the car and the buttons work, and that desktop shows no touch UI. All use the `window.__game` debug handle (mode, health, wasted, `vehicles`, `player`, `city`) exposed from `main.ts` — keep it in sync if you add state worth testing.

## Architecture

The codebase is split along one hard line: **the simulation core is pure and Three.js-free; the rendering/runtime layer owns everything that imports `three`.** This is what makes the gameplay logic unit-testable in a node environment.

**Pure core (no `three`, unit-tested):**
- `src/core/` — `math` (clamp/lerp/frame-rate-independent `damp`/`angleDelta`/`safeApproachSpeed`/`stickVector`/`pursuitSpeed`), `rng` (seeded mulberry32 + `hashSeed` for per-chunk/field seed mixing), `noise` (deterministic simplex fields + `fbm`/`ridged`/`domainWarp`, the basis for the generative world — see `docs/research/generative-world.md`). Browser-glue (not pure, no Three): `Input` (keyboard edge detection), `GameLoop` (fixed-timestep accumulator), `Controls` (merges keyboard + touch into one analog intent).
- `src/audio/RadioModel.ts` — pure tuner logic (station/track index, OFF state, wrap), unit-tested.
- `src/world/City.ts` — deterministic procedural city generation: traffic lanes, streetlights, curbside parking, spawn point, and the `SpatialGrid`. Buildings + colliders come from **`generateChunk(cx,cz,config)`** — a pure function seeded by `hashSeed(seed,cx,cz)`, so a chunk is identical regardless of visit order. The finite city is just a tiling `generateChunk(0,0)..(n,n)` (`config.chunkBlocks` blocks per chunk); this is the seam the streamed world (R007) loads on demand. `src/world/biome.ts` — pure `classify(urbanity, elevation)` + a `BIOMES` data table (density/height/palette per biome), no scattered biome conditionals.
- **World-gen determinism (invariant):** generation is a pure function of `(seed, worldX, worldZ)` — continuous fields (`core/noise`) for anything that crosses a chunk seam, a per-chunk hashed RNG for discrete placement; never `Math.random()`, visit order, or neighbor state. It's unit-tested (same coord ⇒ identical). This is also why a Rust/WASM port is deferred — it would have to match `mulberry32`+noise bit-for-bit.
- `src/vehicles/VehicleModel.ts` — pure arcade vehicle dynamics. `stepVehicle` is a pure function (state + input → state) over a world **velocity vector**; it decomposes velocity into forward/lateral each step and bleeds the lateral part off by tyre grip. The handbrake slashes that grip, which is what produces powerslides.
- `src/systems/Collision.ts` — circle-vs-AABB push-out, circle-vs-circle overlap (for car-on-car), and nearest-point search.

**Render/runtime layer (imports `three`, browser-only):**
- `src/render/` — `Scene` (renderer, dusk lighting, ground/roads, fog), `Assets` (building/car/ped/streetlight mesh factories + material cache), `textures` (procedural facade + radial-glow canvas textures).
- `src/systems/` — `FollowCamera` (smoothed chase cam), `Vehicles` (ALL cars — player, AI traffic, parked-from-`city.parkingSpots` — with one shared physics + collision pass), `Pedestrians` (ambient walkers that get run over), `Debris` (pooled cube gibs flung when a pedestrian is hit), `Smoke` (pooled billboard-**sprite** particles — NOT geometry — that a car under half health trails, thicker as it nears wrecking; owned by `Vehicles`).
- `src/entities/Player.ts` — on-foot avatar controller.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [depixeled-chris/gta7](https://github.com/depixeled-chris/gta7) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-31 -->
