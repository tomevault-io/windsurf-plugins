---
trigger: always_on
description: This document provides project-specific context for AI coding agents, followed by general Obsidian plugin development guidelines.
---

# Obsidian Scatter – Agent Context

This document provides project-specific context for AI coding agents, followed by general Obsidian plugin development guidelines.

---

## Project Summary

**Obsidian Scatter** is a scatter plot view plugin for Obsidian Bases. It plots notes as points on a 2D chart using numeric properties, enabling visual pattern discovery and prioritization.

- **Plugin ID:** `scatter-chart`
- **Entry point:** `src/main.ts` → compiled to `main.js`
- **View type:** Extends `BasesView` to add a scatter plot layout option
- **Required Obsidian version:** 1.10.0+ (Bases View API)

---

## Architecture

### Module Responsibilities

| Module | Responsibility | Purity |
|--------|---------------|--------|
| `types.ts` | Type definitions, constants | Pure (no runtime code) |
| `data.ts` | Extract values, transform points, calculate bounds | Pure functions |
| `render.ts` | Create SVG elements | Pure functions (create new elements) |
| `interactions.ts` | Event handling factories | Returns cleanup functions |
| `scatter-view.ts` | Orchestration, lifecycle | Stateful (required by API) |
| `main.ts` | Plugin registration | Side effects only |

### Data Flow

```
Bases API
    │
    ▼
this.data.groupedData (reactive, replaced on change)
    │
    ▼
flattenGroupedData() → BasesEntry[]
    │
    ▼
extractDataFromEntries() → { points, skipped, categories }
    │
    ▼
calculatePlotBounds() → { x: {min, max}, y: {min, max} }
    │
    ▼
transformPointsToScreen() → RenderedPoint[] (with screenX, screenY, color)
    │
    ▼
createChartSvg() → SVGSVGElement
    │
    ▼
attachPointEvents() → cleanup function
    │
    ▼
DOM (containerEl)
```

### Re-rendering Strategy

- **Current:** Clear container, rebuild everything on `onDataUpdated()`
- **Future:** Diff points, reuse/move existing circles, animate transitions

---

## Key Design Decisions

### Missing Data Handling

**Decision:** Skip entries missing X or Y values. Show count of skipped entries.

**Rationale:** Skipping is the least surprising behavior. Alternatives like defaulting to 0 create false clusters at the origin. The warning count helps users understand what's happening without cluttering the visualization.

### Axis Scaling

**Decision:** Linear scaling, auto-fit to data with 5% padding.

**Rationale:** Auto-fit maximizes visual spread of points. Including zero would waste space when data is clustered in a narrow range. Log scale is a future option for wide-range data.

### Axis Orientation

**Decision:** Small values bottom-left, large values top-right. Standard Cartesian convention.

**Rationale:** Users can create formula properties to invert values if needed. Axis inversion toggle deferred to future versions.

### Point Rendering

**Decision:** Fixed 8px radius, 0.7 opacity, CSS hover transitions.

**Rationale:** Fixed radius keeps MVP simple. Opacity handles overlapping points gracefully. Size-by-property is a future enhancement.

### Color Coding

**Decision:** Optional categorical coloring using Obsidian CSS variables (`--color-blue`, etc.). Cycles through 8 colors. Fallback to muted color for missing categories.

**Rationale:** Theme compatibility out of the box. Color adds information without requiring a third numeric property.

### Rendering Technology

**Decision:** Vanilla SVG with DOM manipulation. No D3 or other chart library.

**Rationale:** SVG is the right fit for up to ~1000 points. Native event handling per element. Keeps bundle small. D3 would add 70KB+ for features we don't need.

### Configuration UI

**Decision:** Use Bases `ViewOption` system for X/Y/Color property selection.

**Rationale:** Integrates with existing "Configure view" menu. Persists to `.base` file automatically.

---

## Performance Targets

| Points | Target Render Time |
|--------|-------------------|
| 100 | instant (<16ms) |
| 500 | smooth (<50ms) |
| 1000 | acceptable (<100ms) |
| 5000+ | consider Canvas or virtualization |

---

## Testing Approach

### Unit Tests

- `data.ts` and `render.ts` contain pure functions → easy to unit test
- Run with `npm test` (uses Vitest)
- Tests live alongside source files: `data.test.ts`, `render.test.ts`

### Manual Testing Checklist

1. Create Base with various filters
2. Test with 0, 1, 10, 100, 500 notes
3. Test with missing properties, non-numeric values
4. Test resize behavior (sidebar, split, full width)
5. Test theme compatibility (light, dark, custom)
6. Test click/hover on overlapping points

---

## Common Development Tasks

### Adding a new view option

1. Add the option to `VIEW_OPTIONS` array in `scatter-view.ts`
2. Read the value from `this.config` in the view
3. Pass it through the data flow to where it's needed

### Changing point styling

1. CSS variables are defined in `styles.css`
2. Point elements are created in `render.ts` → `createPointElement()`
3. Hover effects are in `interactions.ts`

### Modifying data extraction

1. All data extraction happens in `data.ts`
2. Functions are pure: input data, output transformed data
3. Write tests first in `data.test.ts`

### Changing SVG rendering

1. Axis and grid rendering is in `render.ts`
2. Functions return SVG elements, don't mutate external state

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MartinSeeler/obsidian-scatter](https://github.com/MartinSeeler/obsidian-scatter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
