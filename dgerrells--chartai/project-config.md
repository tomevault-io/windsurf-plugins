---
trigger: always_on
description: Guide for LLM agents working with the chartai charting library.
---

# chartai — Agent Reference

Guide for LLM agents working with the chartai charting library.

## Overview

chartai is a GPU-accelerated charting library that renders via WebGPU compute and render pipelines in a web worker. The main thread stays free; charts update passively when needed.

**Architecture:**
- **Main thread:** ChartManager singleton, Chart instances, DOM/canvas setup, plugin UI
- **Web worker:** GPU worker (`gpu-worker.ts`) runs compute shaders and render passes
- **Triple canvas:** back (2D, behind chart), GPU (OffscreenCanvas), front (2D, overlays)
- **Plugins:** Two kinds — **RendererPlugin** (GPU chart types) and **ChartPlugin** (UI: zoom, hover, labels)

**Requirements:** WebGPU with compute shader support. No fallback.

---

## Core API

### ChartManager (singleton)

```ts
import { ChartManager } from 'chartai';
```

| Method | Signature | Description |
|--------|-----------|-------------|
| `use` | `(plugin: RendererPlugin \| ChartPlugin) => void` | Register a chart type or UI plugin. Call before `init()`. |
| `init` | `() => Promise<boolean>` | Start GPU worker. Returns `true` if WebGPU is available. |
| `create` | `(config: ChartConfig & ...) => Chart` | Create a chart. Requires `init()` first and a matching renderer via `use()`. |
| `setTheme` | `(dark: boolean) => void` | Toggle dark mode. |
| `setSyncViews` | `(sync: boolean) => void` | Sync pan/zoom across all charts. |
| `onStats` | `(cb: (stats: ChartStats) => void) => () => void` | Subscribe to FPS/render stats. Returns unsubscribe. |
| `getStats` | `() => ChartStats` | Current stats snapshot. |

### Chart (instance)

Returned by `ChartManager.create()`.

| Method | Signature | Description |
|--------|-----------|-------------|
| `id` | `string` | Chart ID (readonly). |
| `setData` | `(series: ChartSeries[]) => void` | Replace all series data. |
| `configure` | `(patch: Partial<Config>) => void` | Update config, uniforms, bgColor. Triggers render. |
| `addPlugin` | `(plugin: ChartPlugin) => void` | Add UI plugin to this chart only. |
| `removePlugin` | `(name: string) => void` | Remove plugin by name. |
| `hasPlugin` | `(name: string) => boolean` | Check if plugin is installed. |
| `resetView` | `() => void` | Animate back to default pan/zoom. |
| `destroy` | `() => void` | Remove chart and cleanup. |

---

## API with Defaults

### ChartConfig (base, required)

| Field | Type | Required | Default |
|-------|------|----------|---------|
| `type` | `string` | yes | — |
| `container` | `HTMLElement` | yes | — |
| `series` | `ChartSeries[]` | yes | — |
| `defaultBounds` | `{ minX?, maxX?, minY?, maxY? }` | no | auto from data |
| `bgColor` | `[r, g, b]` 0–1 | no | theme-based |

### ChartSeries

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| `label` | `string` | yes | — |
| `color` | `ChartColor \| string` | yes | `{ r, g, b }` 0–1 or any CSS color (hex, rgb, oklab, etc.) |
| `x` | `number[]` | yes | Sorted by X before upload |
| `y` | `number[]` | yes | Same length as `x` |
| `[key]` | `number[]` | no | Extra per-point data; use in shaders via `{key}-data` binding |

### Built-in chart types and their config

| Type | Config | Defaults |
|------|--------|----------|
| `line` | `LineConfig` | `maxSamplesPerPixel: 10000` |
| `area` | same as line | same |
| `scatter` | `ScatterConfig` | `pointSize: 3` |
| `bar` | `BarConfig` | `maxSamplesPerPixel: 10000` |
| `candlestick` | `CandlestickConfig` | `maxSamples: 10000`, `binSize: 8`, `interval: 0`, `upColor`, `downColor` |
| `boids` | `BoidsConfig` | `radius: 6` |

### Plugin options (merged into create config)

| Plugin | Config | Defaults |
|--------|--------|----------|
| zoom | `zoomMode?: ZoomMode` | `"both"` |
| hover | `showTooltip?: boolean`, `onHover?: (HoverData \| null) => void`, `pillDecayMs?: number`, `formatX?`, `formatY?`, `fontFamily?` | `showTooltip: false`, `pillDecayMs: 60` |
| labels | `textColor?`, `gridColor?`, `fontFamily?`, `labelSize?`, `formatX?`, `formatY?` | theme-based colors, `formatX/Y: String` |

`ZoomMode`: `"both"` | `"x-only"` | `"y-only"` | `"none"`

---

## Custom Charts (RendererPlugin)

A custom chart is a `RendererPlugin` with WGSL shaders. Register with `ChartManager.use(MyChart)` before `create()`.

### RendererPlugin

```ts
interface RendererPlugin {
  name: string;                    // Used as config.type
  shaders: Record<string, string>; // Named WGSL entry points
  passes: PassDef[];
  buffers?: BufferDef[];
  uniforms?: UniformDef[];
  computeBounds?: (series) => { minX, maxX, minY, maxY };
  install?: (chart, el) => void;
  uninstall?: (chart) => void;
}
```

### PassDef

```ts
interface PassDef {
  type: "compute" | "render";
  shader: string;           // Key in shaders map
  bindings: BindingDef[];
  perSeries?: boolean;      // Default true for compute
  dispatch?: (ctx: RenderContext) => { x, y?, z? };  // Compute only
  topology?: string;        // Render: "triangle-list" | "line-list" | etc.
  loadOp?: "clear" | "load";
  blend?: BlendState;
  draw?: (ctx: RenderContext) => number;  // Render: vertex count
}
```

### BindingDef

```ts
interface BindingDef {
  binding: number;
  source: string;   // Built-in or buffer name
  write?: boolean;  // For storage buffers
}
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dgerrells/chartai](https://github.com/dgerrells/chartai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
