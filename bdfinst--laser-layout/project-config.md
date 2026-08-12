---
trigger: always_on
description: Laser Layout is a SvelteKit web app that optimizes 2D part nesting for laser cutting. Users upload SVG or LightBurn (.lbrn2) files, configure material sheet dimensions, and the app runs a genetic algorithm to pack parts efficiently across one or more sheets.
---

# CLAUDE.md

## What This Is

Laser Layout is a SvelteKit web app that optimizes 2D part nesting for laser cutting. Users upload SVG or LightBurn (.lbrn2) files, configure material sheet dimensions, and the app runs a genetic algorithm to pack parts efficiently across one or more sheets.

## Commands

- `npm run dev` — start dev server
- `npm run build` — production build
- `npm run lint` — ESLint (TypeScript + Svelte)
- `npm run format` — Prettier (`format:check` to verify, `lint:fix` to auto-fix lint)
- `npm run check` — TypeScript + Svelte type checking
- `npm test` — run all unit tests (vitest)
- `npx vitest run test/geometry/polygon.test.ts` — run a single test file
- `npx vitest run -t "test name"` — run a single test by name
- `npm run test:e2e` (`npx playwright test`) — e2e tests (builds app, serves on :4173)
- `npm run bench` — nesting compaction benchmark (`bench/nesting-compaction.bench.ts`)

## Pre-commit Hook

Husky runs on every commit: lint-staged (ESLint + `prettier --write` on staged `.ts`/`.svelte`, and `prettier --write` on staged `.js`/`.json`/`.css`/`.md`), then `npm run check` (type checking) and `npm test` (all unit tests). All must pass.

## Architecture

### Data Flow

Upload (SVG/LightBurn) → Parser → `Part[]` → Deduplication → Project Store → Nesting Engine → `NestingResult` → Exporter (SVG/LightBurn)

### Core Types (`geometry/types.ts`)

Everything flows through `Part` (id, name, polygons, plus optional `sourceIndex`, `lockOrientation` (forbid mirroring), `grainConstraint` (restrict to 0/180° rotation for directional material), and `priority` `'required'|'optional'`) and `PlacedPart` (part + x/y/rotation + `mirror`). A `Polygon` is `Point[]`. All internal measurements are in mm; display units (mm/in) are converted at the store boundary.

Per-part flags affect placement: `lockOrientation` keeps the part un-mirrored, `grainConstraint` limits its rotations, and `priority: 'optional'` lets the engine drop copies that don't fit rather than opening a new sheet (`'required'` parts always get a sheet).

### Nesting Pipeline

1. **Parsers** (`parsers/`) extract `Part[]` from uploaded files. The SVG parser handles `<path>`, `<rect>`, `<circle>`, `<ellipse>`, `<polygon>`, `<polyline>` with nested `<g>` transform inheritance. The LightBurn parser handles the `.lbrn2` XML format.
2. **Deduplication** (`geometry/dedup.ts`) identifies geometrically identical parts (within configurable tolerance) and collapses them into unique parts with quantity counts.
3. **Engine** (`nesting/engine.ts`) orchestrates multi-sheet nesting. `nestPartsIterative()` is a generator that fills one sheet at a time, yielding progress per GA generation. Parts that don't fit on the current sheet overflow to the next. `makeOptimizerConfig()` maps `NestingConfig` (incl. the optional `stallWindow`/`stallEpsilon`/`maxGenerations`) to the optimizer config, defaulting `maxGenerations = max(generations * 3, 120)`, `stallWindow = 15`, `stallEpsilon = 0.005`.
4. **Optimizer** (`nesting/optimizer.ts`) runs a genetic algorithm where each individual encodes part rotation angles + placement order. Fitness (lower is better) = `unplacedCount·PENALTY + openAreaRatio + gravityWeight·gravity + remnantWeight·(1−remnantRatio) + tiny·stripHeight` — minimizing open area / material waste, feasibility dominating. The GA stops on convergence (`hasStalled()`: no meaningful relative improvement over `stallWindow` generations) bounded by `maxGenerations`, rather than a fixed count.

   **Remnant-aware terms (#41).** Two small, tunable terms shape _where_ the leftover space lands so the offcut is reusable, not scrap. `gravity` (in [0,1], from `stats.gravityMetric`) is the area-weighted part centroid's distance from the (0,0) corner over the sheet diagonal — a mild pull that clusters parts so slack consolidates. `remnantRatio` (from `stats.remnantStats`) is the largest axis-aligned empty rectangle (coarse-grid histogram scan over part bounding boxes) as a fraction of sheet area; fitness rewards it via `1 − remnantRatio`. Both weights (`GRAVITY_WEIGHT`/`REMNANT_WEIGHT`, default `0.05`) are small relative to `openAreaRatio` so density and feasibility stay dominant, and are configurable via `NestingConfig.{gravityWeight,remnantWeight}` (0 disables a term). The terms are opt-in by presence in `fitnessFromStats` — omitting a metric contributes exactly 0, keeping legacy callers/baselines unchanged.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bdfinst/laser-layout](https://github.com/bdfinst/laser-layout) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
