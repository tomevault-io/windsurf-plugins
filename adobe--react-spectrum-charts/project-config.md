---
trigger: always_on
description: Declarative React library for composing Spectrum data visualizations. Charts are described as JSX component trees; the library converts them to Vega specs at runtime.
---

# react-spectrum-charts

Declarative React library for composing Spectrum data visualizations. Charts are described as JSX component trees; the library converts them to Vega specs at runtime.

---

## Package Structure

```
packages/
  react-spectrum-charts/       # Public React component library (@adobe/react-spectrum-charts)
  vega-spec-builder/           # Vega spec generation logic (@spectrum-charts/vega-spec-builder)
  vega-spec-builder-s2/        # Spectrum 2 variant of the spec builder
  react-spectrum-charts-s2/    # Spectrum 2 React component variant
  constants/                   # Shared string constants (signal names, data field names, defaults)
  themes/                      # Color schemes and runtime token resolution
  utils/                       # Shared utility functions
  locales/                     # i18n strings
  dev-tools/                   # Internal contributor CLI tools (not published)
  mcp/                         # MCP server
```

The two packages touched for most feature work:
- **`vega-spec-builder`** — spec logic, types, unit tests
- **`react-spectrum-charts`** — React components, Storybook stories, integration tests

---

## Architecture Reference

For deep architecture context — the VegaChart rendering cycle, signal system, data sources, scale system, interactive mark wiring, COMPONENT_NAME, sanitize gate, encoding conventions, and S2 parity rules — read `.claude/architecture.md`.

---

## The Three-Layer Pipeline

```
React JSX
  ↓  childrenAdapter.ts  — walks the React tree, extracts props into ChartOptions
ChartOptions
  ↓  buildSpec()         — packages/vega-spec-builder/src/chartSpecBuilder.ts
Vega Spec (JSON)
  ↓  vega-embed          — Chart.tsx renders via vega-embed
Rendered chart
```

**Key entry points:**
- `buildSpec(chartOptions)` — `vega-spec-builder/src/chartSpecBuilder.ts` — produces a complete Vega spec from a `ChartOptions` object
- `rscPropsToSpecBuilderOptions()` — `react-spectrum-charts/src/rscToSbAdapter/childrenAdapter.ts` — converts the React component tree to `ChartOptions`
- `Chart.tsx` — `react-spectrum-charts/src/Chart.tsx` — top-level component, calls `useSpec()` → `buildSpec()`

---

## Type System

Every mark follows the same three-type pattern, using Line as the canonical example:

| Type | Location | Purpose |
|---|---|---|
| `LineOptions` | `vega-spec-builder/src/types/marks/lineSpec.types.ts` | All fields optional. Public contract — input to `buildSpec`. |
| `LineSpecOptions` | same file | `LineOptions` with required defaults applied + internal fields (`idKey`, `index`, `colorScheme`, etc.). Used inside spec builder only. |
| `LineProps` | `react-spectrum-charts/src/types/marks/line.types.ts` | React component props. Extends `LineOptions` via `Omit` — replaces child-type fields with `children?: ReactElement`, callbacks with `onClick?` etc. |

`LineOptionsWithDefaults` (in `lineSpec.types.ts`) is a union of string literals naming every field that has a runtime default. It drives the `PartiallyRequired` utility type used by `LineSpecOptions`.

---

## `addLine` — How a Mark Gets Built

`addLine` in `lineSpecBuilder.ts` is the `produce`-wrapped function that mutates the spec for a single line mark. It:
1. Applies all defaults and assembles `LineSpecOptions`
2. Calls `addData()` — data sources, time transforms, tooltip/popover data, trendlines, metric ranges
3. Calls `addSignals()` — hover/select signals, tooltip signals
4. Calls `setScales()` — x/y scales, color/lineType/opacity scales
5. Calls `addLineMarks()` — the mark group, voronoi hover overlay, static points, trendline marks

Each of those four functions is exported and tested independently.

---

## Key Constants (`packages/constants/constants.ts`)

| Constant | Value | Purpose |
|---|---|---|
| `TABLE` | `'table'` | Root Vega data source name |
| `FILTERED_TABLE` | `'filteredTable'` | Filtered/transformed data source |
| `MARK_ID` | `'rscMarkId'` | Unique row ID added by identifier transform |
| `SERIES_ID` | `'rscSeriesId'` | Series identifier field |
| `HOVERED_ITEM` | `'hoveredItem'` | Signal name suffix for hover state |
| `HOVERED_SERIES` | `'hoveredSeries'` | Signal name suffix for hovered series |
| `COLOR_SCALE` | `'color'` | Ordinal color scale name |
| `LINE_TYPE_SCALE` | `'lineType'` | Ordinal line type scale |
| `OPACITY_SCALE` | `'opacity'` | Ordinal opacity scale |
| `DEFAULT_METRIC` | `'value'` | Default y-axis data field |
| `DEFAULT_TIME_DIMENSION` | `'datetime'` | Default x-axis data field |

---

## Adding a Property to an Existing Mark

Always implement in this order:

1. **`vega-spec-builder/src/types/marks/lineSpec.types.ts`**
   - Add the field to `LineOptions` (optional, with JSDoc)
   - Add its key to `LineOptionsWithDefaults` only if it has a runtime default

2. **`vega-spec-builder/src/line/lineSpecBuilder.ts`**
   - Destructure the new field (with its default if any) inside `addLine`'s produce callback
   - Add it to the assembled `lineOptions` object
   - Implement the vega spec change in whichever of `addData`, `addSignals`, `setScales`, `addLineMarks` is affected

3. **`vega-spec-builder/src/line/lineTestUtils.ts`**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [adobe/react-spectrum-charts](https://github.com/adobe/react-spectrum-charts) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
