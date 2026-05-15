---
trigger: always_on
description: Cursor rules for Semiotic data visualization library with 30+ chart types, MCP server, and AI-assisted chart generation.
---

# Semiotic — AI Assistant Guide

## Quick Start
- Install: `npm install semiotic`
- Import: `semiotic`, `semiotic/xy`, `semiotic/ordinal`, `semiotic/network`, `semiotic/geo`, `semiotic/realtime`, `semiotic/ai`, `semiotic/data`, `semiotic/server`
- CLI: `npx semiotic-ai [--schema|--compact|--examples|--doctor]`
- MCP: `npx semiotic-mcp`
- Every HOC has a built-in error boundary (never blanks the page) and dev-mode validation warnings

## Architecture
- **HOC Charts**: Simple props, sensible defaults. **Stream Frames**: Full control.
- **Always use HOC charts** (`ForceDirectedGraph`, `SankeyDiagram`, `LineChart`, `RealtimeLineChart`, `ChoroplethMap`, etc.) unless you need sophisticated control they don't expose. Stream Frames (`StreamNetworkFrame`, `StreamXYFrame`, `StreamOrdinalFrame`, `StreamGeoFrame`) are low-level escape hatches — they accept raw `RealtimeNode`/`RealtimeEdge` wrappers in callbacks, not your data objects directly.
- Every HOC accepts `frameProps` to pass through. TypeScript `strict: true`.

## Common Props (all HOCs)
`title`, `width` (600), `height` (400), `responsiveWidth`, `responsiveHeight`, `margin`, `className`, `enableHover` (true), `tooltip` (boolean | `(datum) => ReactNode` | config object), `showLegend`, `showGrid` (false), `frameProps`, `onObservation` (callback, see below), `chartId`, `loading` (false), `emptyContent`, `legendInteraction` ("none"|"highlight"|"isolate"), `legendPosition` ("right"|"left"|"top"|"bottom", default "right"), `emphasis` ("primary"|"secondary")

### tooltip
`tooltip` accepts: `true` (default tooltip), `false` (disabled), a **function** `(datum: Record<string, any>) => ReactNode`, or a config `{ fields?: string[], title?: accessor, format?: fn, style?: CSSProperties }`. The function form receives your raw data object directly.

### onObservation
`onObservation` receives a `ChartObservation` with `type` and event-specific fields:
- **hover**: `{ type: "hover", datum: <your data>, x, y, timestamp, chartType, chartId }`
- **hover-end**: `{ type: "hover-end", timestamp, chartType, chartId }`
- **click**: `{ type: "click", datum: <your data>, x, y, timestamp, chartType, chartId }`
- **brush**: `{ type: "brush", extent: { x: [min, max], y: [min, max] }, timestamp, chartType }`
- **selection**: `{ type: "selection", selection: { name, fields }, timestamp, chartType }`

The `datum` field contains your original data object (not a wrapper).

## XY Charts (`semiotic/xy`)

**LineChart** — `data`, `xAccessor` ("x"), `yAccessor` ("y"), `lineBy`, `lineDataAccessor` ("coordinates"), `colorBy`, `colorScheme`, `curve`, `lineWidth` (2), `showPoints`, `pointRadius` (3), `fillArea`, `areaOpacity` (0.3), `anomaly` (AnomalyConfig), `forecast` (ForecastConfig), `directLabel` (boolean|{position,fontSize}), `gapStrategy` ("break"|"interpolate"|"zero"), `xScaleType` ("linear"|"log"), `yScaleType` ("linear"|"log")

**AreaChart** — LineChart props + `areaBy`, `y0Accessor` (band/ribbon), `gradientFill` (boolean|{topOpacity,bottomOpacity}), `areaOpacity` (0.7), `showLine` (true)

**StackedAreaChart** — flat array data + `areaBy` (required, groups into stacked areas), `colorBy`, `normalize` (false). Do NOT use `lineBy` or `lineDataAccessor` — those are LineChart props.

**Scatterplot** — `data`, `xAccessor`, `yAccessor`, `colorBy`, `sizeBy`, `sizeRange`, `pointRadius` (5), `pointOpacity` (0.8), `marginalGraphics`

**BubbleChart** — Scatterplot + `sizeBy` (required), `sizeRange` ([5,40]), `bubbleOpacity` (0.6)

**ConnectedScatterplot** — `data`, `xAccessor`, `yAccessor`, `orderAccessor` (number|Date field for sequencing), `pointRadius` (4). Viridis colored start→end, line width = point radius, white halo under lines when <100 points.

**QuadrantChart** — Scatterplot divided into four labeled, colored quadrants. `data`, `xAccessor`, `yAccessor`, `quadrants` (required: `{ topRight, topLeft, bottomRight, bottomLeft }` each with `label`, `color`, optional `opacity`), `xCenter` (vertical center line in data units), `yCenter` (horizontal center line), `centerlineStyle` (`{ stroke, strokeWidth, strokeDasharray }`), `showQuadrantLabels` (true), `quadrantLabelSize` (12), `colorBy`, `sizeBy`, `sizeRange`, `pointRadius` (5), `pointOpacity` (0.8). Supports push API. Quadrant fills and labels drawn via `canvasPreRenderers`.

**Heatmap** — `data`, `xAccessor`, `yAccessor`, `valueAccessor`, `colorScheme` ("blues"|"reds"|"greens"|"viridis" or custom), `showValues`, `cellBorderColor`. Accessors can be string field names (including string/categorical fields) or functions.

## Ordinal Charts (`semiotic/ordinal`)

**BarChart** — `data`, `categoryAccessor`, `valueAccessor`, `orientation`, `colorBy`, `sort`, `barPadding` (40)
**StackedBarChart** — + `stackBy` (required), `normalize`, `barPadding` (40)
**GroupedBarChart** — + `groupBy` (required), `barPadding` (60)
**SwarmPlot** — `data`, `categoryAccessor`, `valueAccessor`, `colorBy`, `sizeBy`, `pointRadius`, `pointOpacity`
**BoxPlot** — + `showOutliers`, `outlierRadius`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [XD3an/awesome-ai-coding-all-in-one](https://github.com/XD3an/awesome-ai-coding-all-in-one) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
