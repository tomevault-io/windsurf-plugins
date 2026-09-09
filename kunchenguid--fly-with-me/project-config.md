---
trigger: always_on
description: The engine's rules, for agents and anyone changing `src/`. Most of them were learned by breaking something. The other docs each own one thing: `VISION.md` is what the page is for, the experience it is meant to create, and the test for any change; `CONTRIBUTING.md` holds every command (serve, bundle, checks, bench, capture, publish) and the scenery contract's prose; `docs/perf-notes.md` holds what a frame costs and what is left to win; `README.md` is the front door, with how to use the page and p
---

# Fly With Me

The engine's rules, for agents and anyone changing `src/`. Most of them were learned by breaking something. The other docs each own one thing: `VISION.md` is what the page is for, the experience it is meant to create, and the test for any change; `CONTRIBUTING.md` holds every command (serve, bundle, checks, bench, capture, publish) and the scenery contract's prose; `docs/perf-notes.md` holds what a frame costs and what is left to win; `README.md` is the front door, with how to use the page and pointers. Do not repeat one file in another.

## Layout

A Three.js page with no assets and no dependencies. `index.html` is the page and the import map; `src/main.js` is the engine, `src/noise.js` the CPU noise, and `src/water.js`, `src/milky-way.js` and `src/galaxy-matter.js` the approved water and night sky; `library/` is the scenery (`contract.js`, `index.js`, one file per biome, species, ruin type and prop). The browser loads them as modules from a static server. `tools/bundle.mjs` folds it all into `dist/index.html`, which `.github/workflows/pages.yml` publishes; `dist/` is generated and ignored, never edited. The bundler folds only static `import ... from` and `export const|let|function|class|default`, so keep modules to those forms, with no dynamic import or `import.meta` and top-level await only in the entry, and run the checks on the source page and on the bundle after an engine change. Keep every path relative so the page works under a project subpath. Three.js is pinned to r185.1 in the page's head; an upgrade is deliberate work with a visual diff, because the WebGPU renderer and TSL still rename things between monthly releases. `tools/` and `tests/` never ship. `assets/` holds the README's hero image only, and neither the bundler nor Pages touches it. `media/`, `review.html`, `review-assets/` and `.lavish/` are local review surfaces, gitignored.

## Terrain and climate

- The CPU heightfield is the only terrain truth; the GPU only reads it. Anything that needs a height (bird, camera, props, sound) reads `heightAt` from that window, interpolating the exact rendered triangle, not bilinearly across a quad, or props float on steep terrain.
- Biome color belongs to the actual triangle, not an interpolated colored corner: the indexed main grid reconstructs its triangle samples in the fragment shader (older studies use `fc`). Shore masks use actual fragment height to avoid triangular beach teeth. When `colorNode` reads a color attribute, leave `vertexColors` off.
- One climate rule: `biomeWeights` on the CPU and the terrain `colorNode` on the GPU are the same soft cells, measured from the nearest cell before the exponential so single precision never underflows. Anything that needs a biome reads `biomeAt`; placement reads the tree cell's center, so a tree's species must have weight in its cell's mix, not at its base. The three field seeds are hashed from the seed, not sliced from its bits.
- Placement RNG stays independent from asset-detail RNG. Grass bending reads immutable geometry-attribute height, never an already transformed `positionLocal.y`.

## Sky, light and time

- The day clock (`dayPhase`, `DAY_SECONDS`) and the sun's clock (`solar`) differ: `solar` runs at one pace by day and faster through a night that is `NIGHT_SHARE` of the cycle, blended over the twilights. Palette keys, `skyBodies`, and the sky events' spans and weights are all in solar phase; anything keyed to the sun's height reads `solar(dayPhase)` or the bodies, never the day clock. `INTRO.stretch` is a day-clock rate on top of that pace.
- Sky and fog share `horizonTint`, including its directional warmth and cloud crossing color; day palettes own both, and changing only a fog uniform breaks the boundary. Gentle local air gives way to complete fog before the streamed terrain ends; height fog and soft cloud volumes cover cloud crossings. Above the deck (`uAbove`) the tint is a haze that keeps the day's hue and its sunward warmth, never a flat cloud white, or the sun vanishes into it; only the whiteout goes white. The cloud sea is lit from a normal taken by finite differences of its own heap, on a ramp that follows the sun's height so it stays sculpted at noon, and the far sea fades into `horizonTint` like the sky.
- `uSunDir` is always the true sun, even below the horizon. One directional light is the sun by day and the moon by night; it may change direction only while its intensity is zero, and it starts only once the disc reaches the horizon, or trees throw shadows uphill from a sun below it.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kunchenguid/fly-with-me](https://github.com/kunchenguid/fly-with-me) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
