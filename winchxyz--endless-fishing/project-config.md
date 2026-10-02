---
trigger: always_on
description: Real-time WebGL2 ocean/fishing game. Physically-based sky driven by a live astronomical
---

# CLAUDE.md — Endless Fishing

Real-time WebGL2 ocean/fishing game. Physically-based sky driven by a live astronomical
ephemeris, spectrum-driven Gerstner ocean, hand-written buoyancy solver.

---

## Stack

| Concern | Choice |
|---|---|
| Build | Vite 8 + TypeScript 5 (`strict`, `noUncheckedIndexedAccess`) |
| Renderer | **three.js `WebGLRenderer` (WebGL2)** — see `DECISIONS.md` §1 |
| Post | `postprocessing` (pmndrs) `EffectComposer` |
| Shaders | `.glsl` / `.vert` / `.frag` files via `vite-plugin-glsl`. **Never** template strings in `.ts` |
| Debug | `lil-gui` (toggle `~`), `stats.js` |
| Tests | `vitest` (node env, pure math only — no WebGL in tests) |
| E2E / capture | `playwright` via `npm run verify` |
| Physics | Hand-written. No physics library. |

---

## Commands

```bash
npm run dev        # vite dev server
npm run build      # tsc --noEmit && vite build   (must be 0 errors, 0 warnings)
npm run preview    # serve dist/
npm run test       # vitest watch
npm run test:run   # vitest run  (CI)
npm run assets     # download + verify CC0 assets into assets/
npm run textures   # process downloaded textures -> KTX2 / ORM packing
npm run verify     # playwright: boot, assert zero console errors, capture screenshots, log FPS
npm run probe      # playwright: capture chosen moments, photometry, helm, horizon zooms
npm run playtest   # playwright: play the loop with the real keys and assert it closes
npm run smoke      # playwright: load the DEPLOYED site and prove it boots
npm run media      # regenerate docs/media/ — the stills and the storm GIF the README shows
npm run lint       # tsc --noEmit
```

---

## Architecture

```
src/
  core/      Engine, Renderer, Loop, Input, Time, ResourceManager, Settings
  astro/     SolarPosition, LunarPosition, SiderealTime, StarCatalog, Refraction, AstroTime
  world/     Ocean, Sky, Atmosphere, Weather, Clouds, Rain, Islands, Tides, Props
  entities/  Boat, Fish, FishingRod, Bobber, Birds, Wake (ribbon + bow spray)
  gameplay/  FishingSystem, CatchTable, Inventory, Progression, Journal
  render/    PostFX, Materials, LODManager, EnvironmentProbe, CSM
  shaders/   *.vert *.frag *.glsl
  ui/        HUD, Journal, Settings (HTML/CSS overlay)
  math/      Gerstner, Noise, PRNG
scripts/     fetch-assets.ts, process-textures.ts, verify.ts
assets/      git-ignored; populated by `npm run assets`
```

### Layering rules (enforce these in review)

1. `src/math/**` and `src/astro/**` are **pure**: no `three` import, no DOM, no globals.
   They are the only things unit-tested and they must stay trivially testable.
2. `src/core/**` may import `three` but knows nothing about the game.
3. `src/world`, `src/entities`, `src/render` may import `core`, `math`, `astro`.
4. `src/ui/**` never imports `three`. It receives a plain data snapshot each frame.
5. No system reaches into another system's internals. Systems read the frozen
   `WorldState` snapshot produced once per frame by `Engine`.

### The single sources of truth

There are exactly four, and duplicating any of them is a bug:

| Thing | Owner | Mirrored in |
|---|---|---|
| Wave displacement | `math/Gerstner.ts` | `shaders/lib/gerstner.glsl` — kept in sync by `test/gerstner.parity.test.ts` |
| Sky/celestial geometry | `astro/*` (pure) | nothing — every renderer *consumes* `EphemerisState` |
| Wind vector | `world/Weather.ts` | consumed by ocean spectrum, flags, clouds, rain, spray, birds, drift |
| Quality knobs | `core/Settings.ts` | systems subscribe to `settings.onChange` and rebuild |

If you add a wave term to the GLSL, you **must** add it to the TS and the parity test
will tell you. A boat that floats through a wave is a hard failure.

---

## Code conventions

- ES modules, named exports. Default exports only for the top-level `main.ts` entry.
- `strict: true`, `noUncheckedIndexedAccess: true`, `exactOptionalPropertyTypes: true`.
  Index access returns `T | undefined` — handle it, do not `!` your way out.
- No `any`. Use `unknown` + a narrowing guard.
- Angles: **radians everywhere internally.** Functions taking or returning degrees are
  suffixed `Deg` (`altitudeDeg`). Astronomy literature is in degrees, so `astro/` converts
  at its boundary and exposes radians.
- Time: internally always UTC + Julian Day (`number`). Never pass a local `Date` across a
  module boundary; pass a `JulianDate`.
- Units: SI. Metres, seconds, kilograms, radians, lux, kelvin.
- Allocation: **zero allocation in the frame loop.** Scratch `Vector3`/`Quaternion`
  instances are module-level `const` and reused. `update(dt)` must not `new` anything.
- Disposal: every class that creates GPU resources implements `dispose()`, and its owner
  calls it. `ResourceManager` tracks and asserts on leak in dev.
- Naming: `PascalCase` classes/types, `camelCase` values, `SCREAMING_SNAKE` module consts.
- Files are one concept each. A file over ~450 lines is a smell; split it.

### Shader conventions

- Shared GLSL lives in `src/shaders/lib/*.glsl` and is `#include`d (vite-plugin-glsl).
- Uniform names match the TS field name exactly (`uSunDirection` <-> `uniforms.uSunDirection`).
- All custom uniforms are prefixed `u`. Varyings `v`. Attributes `a`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [winchxyz/endless-fishing](https://github.com/winchxyz/endless-fishing) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
