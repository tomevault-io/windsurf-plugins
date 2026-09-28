---
trigger: always_on
description: This file is read automatically by Claude Code at the start of every session. It is the short, authoritative guide to how this project is built and changed. Longer explanations live in `docs/`. **New session? Read `docs/HANDOFF.md` first**: the owner's preferences, decisions that were tried and reversed, and how to ship.
---

# CLAUDE.md: working on gcdatlas

This file is read automatically by Claude Code at the start of every session. It is the short, authoritative guide to how this project is built and changed. Longer explanations live in `docs/`. **New session? Read `docs/HANDOFF.md` first**: the owner's preferences, decisions that were tried and reversed, and how to ship.

## Start here

State on 2026-09-25: **v0.7.5 is live** and `main` is in sync, so the next release is **v0.7.6**. Read `docs/HANDOFF.md`, which covers the owner's preferences, decisions not to undo, how the work is done and the open ideas. Then run `npm install` (first time only) and `npm test`. Use git and `gh` directly: branch `release/vX.Y.Z`, open a PR, check the Vercel preview, then merge with a merge commit.

## What this is

gcdatlas (https://gcdatlas.vercel.app, repo `eshin087/gcdatlas`) is a single-page WebGL2 atlas of the universe rendered entirely as ASCII characters. Real positions, distances and sizes; physically based, artistic rendering. It is meant to be a long-term project that keeps growing (more objects, an ASCII Earth, social features later) without breaking what exists.

## Golden rules

1. **Never break the build or the page.** Run `node build.mjs` (it syntax-checks the bundle) and `npm test` before every commit. Look at screenshots of anything visual you touched (`npm run shots -- key:view`).
2. **One self-contained page.** No runtime frameworks, no bundler, no external JS/CSS at runtime (Google Fonts is the only exception, and the page must still work without it). Everything is concatenated by `build.mjs`.
3. **New features ship behind a flag** (`FLAGS` in `src/04-world.js`, see `docs/FEATURE_FLAGS.md`) when they are experimental, touch the network, or could be scrapped. Default experimental flags to `false`.
4. **Performance is a feature.** Keep 60 fps on a mid-range laptop. Objects compile shaders lazily, CPU sims only run while visible (`sim` + `simActive`), far objects are single impostor dots. Never add per-frame work that scales with the total object count beyond a cheap loop.
5. **Honesty about accuracy.** Real data stays real; anything illustrative, magnified, sped up or simulated says so in its readout or fact. Update `docs/ACCURACY.md` when you add something that is not measured data.
6. **Privacy.** Nothing about the visitor leaves their device. Settings, location, collection log and daily streak live in `localStorage` under `gcdatlas.*`. The only network calls are the fonts and our own `/api/*` functions.
7. **Writing style for anything a visitor reads**: short, concrete, true sentences; no em dashes; plain words over jargon; numbers with units.

## Commands

```sh
node build.mjs                 # build dist/index.html (+ dist/artifact.html); fails on syntax errors
npm test                       # smoke test + phone layout test (tests/smoke.mjs, tests/mobile.mjs)
npm run test:mobile            # just the phone layout (390 x 844 and 844 x 390)
npm run test:tour              # long tour regression (tests/tour.mjs)
npm run shots -- sun:0,crab:1  # screenshots into tests/out/ (+ a contact sheet)
npm run catalog                # regenerate docs/CATALOG.md from the built page
npx vercel dev                 # local server with the /api functions
```

Tests need `npm install` once (dev dependencies: playwright, sharp). Headless Chromium runs WebGL through SwiftShader, which is slow: simulations run at a lower frame rate in tests, so use `__cosmos.simulate(seconds)` or step `o.update()` for deterministic checks.

## Architecture in one screen

- `build.mjs` concatenates `src/**/*.js` in rank order into one IIFE (opened in `02-core.js`, closed in `99-close.js`). Order: core (02) → GLSL (03) → world (04) → data (05) → sky/galaxies (06*) → objects (`o*.js`, then `src/objects/*`) → extras/music (07*) → camera/tours (08*) → render/UI (09*) → close (99).
- Rendering: each object is a ray-marched volume inside a bounding sphere (`prog`, drawn by `drawVolume`) and/or particle systems (`particles`). The scene renders to an HDR buffer at 2x the character grid; `FS_CELL` picks a glyph per cell; a blur adds glow; `FS_FINAL` composites glyphs. Opaque unlit cells are marked void (no glow bleeds into black hole shadows).
- Camera: focus-relative (the camera position is stored relative to the object it looks at), so light-years and metres coexist. Flights use van Wijk–Nuij paths. `SKYV` switches to the planetarium camera.
- Objects: `addObj({...})` and helpers (`addStar`, `addBody`, `addGalaxy`, `namedStar`, `addProbe`). Required: `key`, `name`, `type`, `group`, `pos` (light-years, galactic frame) or `parent`+`offset`, `rad`, `views`. See `docs/ARCHITECTURE.md` → "Object definition".
- UI state: `SET` (settings, persisted), `FLAGS` (feature flags), `tour`, `orbit`, `cam`. Test hooks on `window.__cosmos`.

## Adding content (the most common task)

1. Check `docs/CATALOG.md` (generated) and the backlog in `docs/CONTENT.md` so you do not duplicate an object.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [eshin087/gcdatlas](https://github.com/eshin087/gcdatlas) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-26 -->
