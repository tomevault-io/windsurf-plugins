---
trigger: always_on
description: `function-plot` is a 2D function plotting library for the web powered by D3.js. It supports multiple function representations (linear, implicit, parametric, polar, vectors, points), multiple graph types (polyline, interval arithmetic, scatter, text, annotations), derivative/secant overlays, interactive zoom/pan, and multi-threaded evaluation via web workers.
---

# Agents Guide for `function-plot`

`function-plot` is a 2D function plotting library for the web powered by D3.js. It supports multiple function representations (linear, implicit, parametric, polar, vectors, points), multiple graph types (polyline, interval arithmetic, scatter, text, annotations), derivative/secant overlays, interactive zoom/pan, and multi-threaded evaluation via web workers.

---

## Design guidance for contributors and agents

Before proposing or implementing API-level changes, read these documents in
order:

1. [`design/README.md`](design/README.md) — design-document index and reading
   order.
2. [`design/vision.md`](design/vision.md) — repository-level philosophy and
   durable design principles.
3. [`design/api-refactor.md`](design/api-refactor.md) — the concrete `main`/2.0
   API shape, contracts, lifecycle semantics, and implementation seams.

The current `src/` implementation is the legacy v1-style API being migrated.
Its `Chart`, mutable options, D3-selection-based marks, and cached runtime
metadata describe compatibility constraints, not the target design. New API
work must follow the vision and API refactor documents rather than copying
legacy mutation or renderer coupling into the new surface.

---

## Architecture & Codebase Map

### Core Pipeline

The plotting pipeline follows a **Compile $\rightarrow$ Sample/Eval $\rightarrow$ Render** flow:

1. **Instantiation**: `functionPlot(options)` creates or retrieves a cached `Chart` instance (`src/index.ts`, `src/chart.ts`).
2. **Options & Defaults**: Options and data arrays are normalized and validated (`src/datum-defaults.ts`, `src/datum-validation.ts`).
3. **Axes & Scales**: Coordinate scales (linear or logarithmic) and D3 zoom/pan handlers are mounted to the SVG target (`src/chart.ts`).
4. **Sampling / Evaluation**: Each datum is sampled across the domain using one of the available samplers (`src/samplers/`):
   - `interval` (sync) & `asyncInterval` (web worker pool): Uses interval arithmetic to handle discontinuities and asymptotes cleanly.
   - `builtIn`: Uses standard numeric evaluation for smooth curves and points.
5. **Rendering**: Sampled data is converted into SVG elements via graph plotters (`src/graph-types/`).

### Directory Layout

- **`src/`**: TypeScript and ES module source files
  - `index.ts`: Public library entry point, exports `functionPlot`, `Chart`, graph types, and samplers.
  - `chart.ts`: Main `Chart` class managing D3 SVG elements, scales, axes, margins, event dispatching, and render loop.
  - `types.ts`: TypeScript interfaces (`FunctionPlotOptions`, `FunctionPlotDatum`, `FunctionPlotScale`, etc.).
  - `samplers/`:
    - `builtIn.ts`: Standard numeric evaluation using `built-in-math-eval`.
    - `interval.ts`: Interval arithmetic evaluation using `interval-arithmetic-eval`.
    - `interval_worker_pool.ts` / `interval.worker.mjs`: Multi-worker pool using zero-copy `ArrayBuffer` transfer for high-performance async sampling.
    - `eval.mjs`: Sampler registry and dispatch logic.
  - `graph-types/`:
    - `interval.ts`: Renders interval bounds as vertical SVG path rectangles (`M x y v dy`).
    - `polyline.ts`: Renders continuous line segments and closed filled areas.
    - `scatter.ts`: Renders discrete scatter points.
    - `text.ts`: Renders text labels at coordinates.
    - `annotation.ts`: Renders horizontal/vertical guideline annotations.
    - `derivative.ts` / `secant.ts`: Renders tangent and secant helper lines.
  - `tip.ts`: Tooltip interaction on mouseover/mousemove.
  - `perf/`: Performance benchmarking scripts with `tinybench` and `tsx`.
- **`site/`**: Documentation and interactive playground
  - `site/js/examples.js`: Source of truth for interactive documentation examples.
  - `site/js/examples-to-html.cjs`: Script that parses `examples.js` and generates HTML previews using Pug and Markdown-it.
  - `site/docs/`: Generated TypeDoc API documentation.
- **`test/`**:
  - `test/e2e/graphs.test.ts`: Puppeteer + Jest end-to-end visual regression tests using `jest-image-snapshot` (SSIM comparisons).
  - `test/e2e/snippets.ts`: Test cases executed against the headless browser.
- **`design/`**: Contributor-facing vision and API direction plus architecture
  notes and performance benchmarks (`vision.md`, `api-refactor.md`,
  `pipeline.md`, `web-workers.md`).
- **`dist/`**: Built distribution artifacts (UMD bundle, ESM output, TypeScript declarations).

---

## Development Workflows

### Prerequisites

- Node.js: `>= 18.12.0`
- Package Manager: `npm`

### Essential Commands

| Command                    | Purpose                                                                                                              |
| -------------------------- | -------------------------------------------------------------------------------------------------------------------- |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mauriciopoppe/function-plot](https://github.com/mauriciopoppe/function-plot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
