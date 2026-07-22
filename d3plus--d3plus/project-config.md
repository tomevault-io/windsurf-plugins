---
trigger: always_on
description: Guidelines for AI agents working in this repository.
---

# AGENTS.md — d3plus

Guidelines for AI agents working in this repository.

## Project Overview

d3plus is a JavaScript data visualization library that extends D3.js. It is organized as a **pnpm monorepo** with workspaces under `packages/`. Each package is independently publishable under the `@d3plus/` npm scope.

### Packages

| Package | Purpose |
|---|---|
| `@d3plus/color` | Color utilities and defaults |
| `@d3plus/core` | All chart types, components, and shapes |
| `@d3plus/data` | Data manipulation and filtering |
| `@d3plus/dom` | DOM utilities |
| `@d3plus/export` | SVG/image export |
| `@d3plus/format` | Number and date formatting |
| `@d3plus/locales` | Translation dictionaries |
| `@d3plus/math` | Math utilities |
| `@d3plus/render` | Scene-graph renderer abstraction + SVG/Canvas backends |
| `@d3plus/react` | React component wrappers |
| `@d3plus/text` | Text measurement and wrapping |
| `@d3plus/types` | Unified TypeScript type definitions |
| `@d3plus/docs` | Storybook documentation site (private, not published) |

## Architecture (v4)

v4 is a scene-graph rewrite. A chart no longer mutates the DOM as it draws.
Instead it compiles to a **serializable scene graph** that a pluggable backend
paints:

```
config (fluent setters / config())
  → pipeline stages (pure-ish transforms over the viz)
    → scene graph (SceneNode tree from @d3plus/render)
      → renderer backend (SVG default, or Canvas)
```

**The public chart API is unchanged from v3** — fluent setters, `config()`, the
`RESET` token, chart classes, and event handlers all behave as before. The
rewrite is under the hood.

### BaseClass + the fluent API

All visualization classes ultimately inherit from `BaseClass`
(`packages/core/src/utils/BaseClass.ts`): a chainable API where each public
setter returns `this`, deep config merging via `config()`, the `RESET` token,
and locale/event management.

Fluent accessors are **schema-driven** in v4. A chart declares its configurable
properties as `ConfigField`s (see `packages/core/src/fluent.ts`); `installFluent`
installs them as `viz.<key>()` getter/setters backed by `viz.schema.<key>`.
Hand-written accessors still exist where behavior is non-trivial, but most
identity-coerce properties come from the schema.

### Charts are values: `ChartDefinition` + `makeChart`

A chart is a **def value**, not a hand-written class. Each chart lives in its own
folder:

```
packages/core/src/charts/<Chart>/
  index.ts        # the ChartDefinition + `export default makeChart(def)`
  applyLayout.ts  # chart-specific TransformStage (layout math)
  emit.ts         # pure: laid-out data → SceneNode[]
  …               # optional helpers (thresholdFunction.ts, etc.)
```

`ChartDefinition` (`charts/definition/ChartDefinition.ts`) is
`{name, features, fields?, ctx?, layoutStage?, emit?, thresholdFunction?,
chartTransform?, setup?, …}`. `makeChart(def, Base = Viz)`
(`charts/definition/makeChart.ts`) produces the class: its constructor runs
`applyDefinition(this, def)` (seeding `fields`/`ctx` and installing fluent
accessors), and its `_draw` runs the shared Viz pipeline then the chart's own
`layoutStage` via `runChartDraw`.

Two kinds of def:

- **Data-driven** (`emit(ctx) → SceneNode[]`): Treemap, Pack, Pie, Tree,
  Network, Sankey, Rings, Geomap, Matrix, RadialMatrix, Priestley, Radar.
- **Paint-driven** (`paintDriven: true`): the Plot family (BarChart, LinePlot,
  AreaPlot, StackedArea, BoxWhisker, BumpChart). `Plot._paint` (via `plotPaint`)
  builds `viz._chartScene`; there is no `emit` step.

`makeChart`'s `Base` argument lets a chart specialize another: e.g.
`makeChart(barChartDef, Plot)`, `makeChart(donutDef, Pie)`.

### Feature modules

Chart "chrome" (legend, color scale, timeline, title/subtitle/total, back
button, attribution, zoom controls) are **opt-in `FeatureModule`s**
(`charts/features/`). A def lists the features it composes in
(`features: [backFeature, titleFeature, …]`); `runLayout` runs each feature's
`layout()`, which claims margin and returns a panel to compose into the scene.

### The pipeline

The draw flow exists as free functions so it can run without the class
lifecycle:

- `vizPreDraw` / `vizPreDrawPure` — data prep (draw depth, filtered/legend data,
  threshold).
- `vizDraw` / `vizDrawPure` — feature layout + margin/panel assembly.
- `runVizPipeline(viz)` — the full orchestrator (`_preDraw → _draw → features →
  paint`).
- `resolveSpec(viz)` — frozen config/context snapshot (the config↔context
  boundary).

These operate on returned values rather than mutating `this` at the outer layer.

### Rendering backends (`@d3plus/render`)

`@d3plus/render` owns the scene graph (`SceneNode` union: rect/circle/line/area/
path/image/text/group/htmlOverlay), the diff/animate layer, and two backends:
`SvgRenderer` (default, most complete) and `CanvasRenderer` (dense, high-shape
charts; pointer hit-testing via `Path2D`). Every viz accepts
`.renderer("svg" | "canvas")`.

### Package anatomy

```
packages/<name>/
  index.ts          # Re-exports the package's public API from src/
  src/              # Source (TypeScript ES modules)
  es/               # Build output: transpiled ESM (gitignored, generated)
  umd/              # Build output: UMD bundles (gitignored, generated)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [d3plus/d3plus](https://github.com/d3plus/d3plus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
