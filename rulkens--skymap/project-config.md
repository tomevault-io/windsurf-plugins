---
trigger: always_on
description: Quick orientation for an AI agent picking up work here; it points at the deeper docs.
---

# Skymap — Claude onboarding

Quick orientation for an AI agent picking up work here; it points at the deeper docs.

## What this is

A WebGPU 3D galaxy renderer: three real catalogs (SDSS, 2MRS, GLADE) parsed at build time into a custom binary format, loaded in the browser, drawn as instanced point billboards with per-galaxy thumbnail quads on close approach. TS + Vite + React UI shell; raw WebGPU + WGSL renderer.

## Where to look

```
src/
  @types/  one type per file; deep relative imports, no barrels
  components/  React UI shell (InfoCard, SettingsPanel, ScaleBar, StatusBar)
  compositions/  build-time engine compositions (app; reference engines later)
  data/  static data: sources enum, colourIndex spec, binary format
  hooks/  React hooks (useEngine, useSplash, alias/structure indexes, …)
  layers/  per-Layer modules — settings clusters today, whole Layers from PR (d)
  services/
    camera/  OrbitCamera, OrbitControls, tweens
    engine/  engine orchestrator, autoLod, cloud loader
      galaxyGenerator/  galaxy generation: v1/ sprite stars (to be deleted),
                        v2/ analytic field, shared/ — READMEs in each
    gpu/  renderers, texture atlas, image queue/fetcher, WGSL shaders
    input/  SpaceMouse + raw input → camera deltas
  state/  RTK slices/selectors/sagas per domain; forbids react-redux (see store/)
  store/  RTK store wiring: createAppStore, root reducer/saga, effects
  styles/  global.css — design tokens + body/html reset only
  utils/  pure helpers (math, format, random) — heavily tested
tools/
  animation/  tourLength — beat-sheet / clip-length reporting
  catalog/  buildAllBins (pipeline entry), crossMatch dedup, subsampleByAbsMag
  curation/  shared curation helpers (dedupeByProximity, writeMetaSidecar)
  dev/  tmux + worktree helpers — see "Tmux workflow helpers" below
  famous/  famous-galaxy seed expansion + image fetchers
  famous-curator/  hand-curate Famous thumbnails (npm run curate-famous)
  filaments/  buildFilaments — DisPerSE wrapper
  flow/  CF4++ peculiar-velocity flow-field builder + verifier
  flow-workbench/  WebGPU dev tool visualising the flow field
  galaxy-renderer/  dev tool: procedural Hubble-sequence galaxy + HDR bloom
  volumes/  scalar-field volume builders (CF-4, MCPM) + diagnostics
  fonts/  buildFontAtlas — MSDF multi-font atlas generator
  perf/  GPU-timing harness (npm run perf) → tools/perf/README.md
  record/  offline tour recorder → mp4 (npm run record-tour)
  site/  makeFavicon, makeOgImage
  structures/  buildStructures — cluster/supercluster catalog builder
  deploy/  syncR2 + r2Cors.json + r2-static/ static assets
  fetch/  external-catalog fetchers with on-disk resume caches
  parsers/  SDSS CSV, 2MRS fixed-width, GLADE fixed-width, NPY, ND-skeleton
  utils/  tools-only helpers — one file per function, deep imports
  vendor-types/  ambient .d.ts shims for msdf-bmfont-xml and pngjs
data/
  raw/  catalog sources, one subdir per source (2mrs/, glade/, hyperleda/, sdss/,
        cf4/, mcpm/, milliquas/, filaments/, famous/, fonts/, mcxc/, mscc/).
        VizieR ReadMes live beside their files (byte layouts!). Paths go
        through tools/utils/io/rawDataRegistry.ts.
docs/BACKLOG.md  ground-truth list of what's next
docs/superpowers/plans/  active implementation plans; shipped → plans/completed/
docs/superpowers/specs/  design specs; shipped → specs/completed/
tests/  Vitest suite — mirrors src/ tree
```

## Project conventions (these override defaults)

- **Didactic but budgeted comments**: explain _why_, never _what_ the code does. A comment earns its place by recording something a reader would otherwise rediscover the hard way: a landmine (including a choice that looks wrong and would get "fixed" back), a unit, a derivation, a cross-file contract. Budget: **module header ≤ 10 lines, comment lines ≤ half the code lines in the file.** Past the budget the material is either not load-bearing, or it belongs in the spec/plan — link it rather than inlining it. Research surveys, option comparisons and "we tried X first" are plan content; history is the git log's job. (Overrides the default no-comments rule; detail in [`docs/superpowers/conventions/comments.md`](docs/superpowers/conventions/comments.md).)
- **`type` aliases, never `interface`**: `export type X = { ... }` for all TS shapes.
- **No barrel exports for components**: import React components directly from their `.tsx`. No `index.ts` re-export files in component folders.
- **One symbol per file in `utils/` and `@types/`**: every file in `src/utils/` (and `tools/utils/`) exports exactly **one function**; every file in `src/@types/` exports exactly **one type**. Filename = the exported symbol's name (kebab/camel as the symbol dictates). No multi-export helper grab-bags — if a "data" file grows a generic pure helper, extract it to its own `utils/<area>/<fn>.ts` (math/format/color/random/gpu/…) with a focused test. Deep relative imports, no barrels.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rulkens/skymap](https://github.com/rulkens/skymap) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-14 -->
