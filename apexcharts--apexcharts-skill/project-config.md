---
trigger: always_on
description: >
---


# ApexCharts AI Skill

> **Framework wrapper detection — check `package.json` before generating code.**
> - `react` → use **`react-apexcharts`** instead of the core API.
> - `vue` (Vue 3) → use **`vue3-apexcharts`**. Vue 2 → **`vue-apexcharts`**.
> - `@angular/core` → use **`ng-apexcharts`**.
>
> Wrappers handle `destroy()` automatically on unmount, accept reactive props, and forward events as idiomatic framework events. Use the core API directly only when no framework is detected, or when the user explicitly asks for vanilla. See `references/framework-wrappers.md`.

## 1. Critical Rules

1. **Always call `chart.render()`** after `new ApexCharts(el, options)`. The constructor does not render.
2. **Always call `chart.destroy()`** before creating a new chart on the same element. Failing to do so causes memory leaks and duplicate charts (especially in React/Vue).
3. **Series data format is chart-type-specific.** Refer to the Data Format Table below. This is the #1 source of AI mistakes.
4. **`yaxis` must be an array** when using multiple y-axes. Each entry needs a `seriesName` to map to the correct series.
5. **`tooltip.shared` and `tooltip.intersect` are mutually exclusive.** `shared: true` shows all series at an x-position. `intersect: true` shows only the hovered point.
6. **Use `null` (not `undefined` or empty string)** for missing data points. `undefined` is silently ignored and breaks the chart.
7. **`chart.stacked: true`** only works with `bar` and `area` chart types.
8. **Responsive breakpoints** must be in ascending order in the `responsive` array.
9. **For mixed/combo charts**, set `type` on each individual series object, not just on `chart.type`.
10. **RadialBar values must be 0–100** (they represent percentages).
11. **Color hex values must include the `#` prefix** (e.g., `'#FF5733'`, not `'FF5733'`).
12. **Tree-shaking**: importing `apexcharts/core` gives you a bare class — you must also import chart-type entries and feature entries separately.

---

## 2. Series Data Format Table

This is the most critical reference. Using the wrong data format is the #1 cause of broken charts.

### Axis Charts (line, area, bar, scatter, etc.)

| Chart Type | `chart.type` | Series Format | Minimal Example |
|---|---|---|---|
| Line | `'line'` | `[{ name, data: [number \| null] }]` or `[{ name, data: [{ x, y }] }]` | `series: [{ name: 'Sales', data: [30, 40, null, 50] }]` |
| Area | `'area'` | Same as line | `series: [{ name: 'Views', data: [10, 20, 30] }]` |
| Bar / Column | `'bar'` | Same as line. Use `plotOptions.bar.horizontal: true` for horizontal bars (default). Set `horizontal: false` for vertical columns. | `series: [{ name: 'Revenue', data: [44, 55, 41] }]` |
| Scatter | `'scatter'` | `[{ name, data: [{ x, y }] }]` — always use XY format | `series: [{ name: 'Points', data: [{ x: 1, y: 5 }, { x: 2, y: 10 }] }]` |
| Bubble | `'bubble'` | `[{ name, data: [{ x, y, z }] }]` — **z is required** (bubble size) | `series: [{ name: 'Data', data: [{ x: 1, y: 30, z: 10 }] }]` |
| Range Area | `'rangeArea'` | `[{ name, data: [{ x, y: [low, high] }] }]` | `series: [{ name: 'Temp', data: [{ x: 'Jan', y: [5, 15] }] }]` |
| Range Bar | `'rangeBar'` | `[{ name, data: [{ x, y: [start, end] }] }]` — for timeline/Gantt, use timestamps | `series: [{ name: 'Tasks', data: [{ x: 'Design', y: [1, 5] }] }]` |
| Candlestick | `'candlestick'` | `[{ data: [{ x, y: [O, H, L, C] }] }]` — array of 4: Open, High, Low, Close | `series: [{ data: [{ x: new Date('2024-01-01'), y: [51, 56, 48, 53] }] }]` |
| Box Plot | `'boxPlot'` | `[{ data: [{ x, y: [min, Q1, median, Q3, max] }] }]` — array of 5 | `series: [{ data: [{ x: 'Group A', y: [10, 20, 30, 40, 50] }] }]` |
| Heatmap | `'heatmap'` | `[{ name, data: [{ x, y: number }] }]` — y is the intensity value | `series: [{ name: 'Mon', data: [{ x: '10am', y: 45 }] }]` |
| Treemap | `'treemap'` | `[{ data: [{ x, y: number }] }]` — y is the area/value | `series: [{ data: [{ x: 'Item A', y: 100 }, { x: 'Item B', y: 60 }] }]` |
| Radar | `'radar'` | `[{ name, data: [number] }]` + `xaxis: { categories: [...] }` | `series: [{ name: 'Skill', data: [80, 50, 30, 40, 100] }]` |

### Non-Axis Charts (pie, donut, radialBar, polarArea)

These use a **flat number array** for `series`, NOT the object format:

```js
// CORRECT — flat number array + labels
{
  chart: { type: 'pie' },  // or 'donut', 'polarArea', 'radialBar'
  series: [44, 55, 13, 43, 22],
  labels: ['Team A', 'Team B', 'Team C', 'Team D', 'Team E']
}
```

---

## 3. Package / Module Map

```
# Full bundle (all chart types + all features)
import ApexCharts from 'apexcharts'

# Bare core (no chart types, no optional features — must register manually)
import ApexCharts from 'apexcharts/core'

# Per-type entry points (registers specific chart types + core)
import ApexCharts from 'apexcharts/line'         # line, area, scatter, bubble, rangeArea
import ApexCharts from 'apexcharts/area'          # same as /line
import ApexCharts from 'apexcharts/scatter'       # same as /line
import ApexCharts from 'apexcharts/bubble'        # same as /line
import ApexCharts from 'apexcharts/rangeArea'     # same as /line
import ApexCharts from 'apexcharts/bar'           # bar, column, rangeBar
import ApexCharts from 'apexcharts/column'        # same as /bar

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [apexcharts/apexcharts-skill](https://github.com/apexcharts/apexcharts-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
