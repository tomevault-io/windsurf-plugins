---
trigger: always_on
description: Monorepo for generating and rendering handwriting animations from any font.
---

# Tegaki

Monorepo for generating and rendering handwriting animations from any font.

## Tech Stack

- **Runtime**: Bun
- **Language**: TypeScript (strict mode, ESNext, nodenext modules)
- **CLI framework**: [Padrone](https://github.com/KurtGokhan/padrone) - schema-first CLI with Zod v4
- **Font parsing**: opentype.js
- **Linter/Formatter**: Biome (2-space indent, single quotes, 140 line width)
- **Testing**: Bun's built-in test runner
- **Monorepo**: Bun workspaces

## Packages

- `packages/renderer` (`tegaki`) — Published npm package. Framework-agnostic animated handwriting renderer with adapters for React, Svelte, Vue, SolidJS, Astro, Web Components, vanilla JS, and Remotion. Ships pre-generated bundles for Caveat, Italianno, Tangerine, and Parisienne under `tegaki/fonts/*`.
- `packages/generator` (`tegaki-generator`) — Internal CLI + library that generates glyph data from fonts. Not published; users generate font data via the website UI (which calls the same pipeline in-browser).
- `packages/website` (`@tegaki/website`) — Astro + Starlight site containing the docs, framework examples, and the interactive generator/preview app at `/tegaki/generator/`.

## Commands

```bash
bun start          # Run the CLI (generator)
bun dev            # Watch mode (website)
bun run test       # Run tests (all packages)
bun typecheck      # TypeScript checks (all packages)
bun check          # Biome lint + format check
bun fix            # Biome auto-fix
bun checks         # All checks: lint + format + typecheck + tests
```

Use these commands instead of custom commands as much as possible. It's crucial that you don't use `bun run` when running these commands, as these are already whitelisted for agent use.

## Architecture

### Renderer (`packages/renderer`)

The `tegaki` npm package is a framework-agnostic renderer with a shared core engine and thin per-framework adapters. Each adapter is exposed as a subpath export (`tegaki/react`, `tegaki/svelte`, `tegaki/vue`, `tegaki/solid`, `tegaki/astro`, `tegaki/wc`). The bare `tegaki` entry re-exports the React adapter for ergonomic backwards compatibility.

```
packages/renderer/src/
  index.ts                    # Re-exports React adapter
  types.ts                    # Shared types: Point, TimedPoint, BBox, Stroke, TegakiBundle, TegakiEffects, etc.
  core/                       # Framework-agnostic engine
    engine.ts                 # TegakiEngine — timeline, playback, time control, bundle loading
    createBundle.ts           # Builds a TegakiBundle from parts
    bundle-registry.ts        # Global bundle registry (register/lookup by family name)
    render-elements.ts        # Low-level SVG element construction
    types.ts                  # Engine-level types (TimeControlProp, effect config, etc.)
  lib/                        # Shared helpers used by both core and adapters
    timeline.ts               # computeTimeline() — per-grapheme animation schedule
    textLayout.ts             # Line breaking, advance widths, RTL/LTR
    drawGlyph.ts              # Glyph -> SVG path drawing
    drawFallbackGlyph.ts      # Fallback when glyph missing from bundle
    effects.ts                # Glow, wobble, pressureWidth, taper, gradient
    strokeCache.ts            # Memoized stroke subdivision (CSS-pixel aware)
    css-properties.ts         # CSS custom property plumbing for animation state
    font.ts                   # FontFace registration helpers
    utils.ts
  react/TegakiRenderer.tsx    # React adapter
  svelte/                     # Svelte 5 adapter
  vue/                        # Vue 3 adapter
  solid/                      # SolidJS adapter
  astro/TegakiRenderer.astro  # Astro adapter (SSR-capable)
  wc/                         # Web Component adapter (`<tegaki-renderer>`)
  remotion/                   # Remotion-specific helpers
```

Pre-generated font bundles live outside `src/`, under `packages/renderer/fonts/<family>/` and are regenerated via `bun --filter tegaki generate-fonts`.

### Generator (`packages/generator`)

CLI entry point uses Padrone. The `generate` command orchestrates a pipeline that processes each glyph through several stages.

#### Pipeline (per glyph)

```
Font download -> Parse (opentype.js) -> Flatten beziers -> Rasterize -> Skeletonize -> Trace -> Compute width -> Order strokes -> JSON output
```

1. **Extract** (`src/font/parse.ts`): opentype.js extracts path commands and metrics
2. **Flatten** (`src/processing/bezier.ts`): Adaptive de Casteljau subdivision converts bezier curves to polyline segments
3. **Rasterize** (`src/processing/rasterize.ts`): Scanline fill with nonzero winding rule produces a binary bitmap
4. **Skeletonize** (`src/processing/skeletonize.ts`): Reduces the bitmap to a 1px-wide skeleton. Default is Zhang-Suen thinning; the pipeline also supports Guo-Hall, Lee 3D, morphological thin, medial-axis (distance transform ridge), and Voronoi-based medial axis (`voronoi-medial-axis.ts`).
5. **Trace** (`src/processing/trace.ts`): Walks skeleton pixels into polylines, prunes short spurs, simplifies with Ramer-Douglas-Peucker
6. **Width** (`src/processing/width.ts`): Distance transform computes stroke width (diameter) at each skeleton point

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [KurtGokhan/tegaki](https://github.com/KurtGokhan/tegaki) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-19 -->
