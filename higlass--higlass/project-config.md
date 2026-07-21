---
trigger: always_on
description: A web-based viewer for large genomic/Hi-C datasets that supports synchronized multi-view navigation with GPU-accelerated rendering. Live at https://higlass.io.
---

# HiGlass — Agent Reference

## What is HiGlass?

A web-based viewer for large genomic/Hi-C datasets that supports synchronized multi-view navigation with GPU-accelerated rendering. Live at https://higlass.io.

## Project Basics

- **License**: MIT
- **Node**: 22.x recommended
- **Package**: Published to npm as `higlass`

## Tech Stack

| Layer        | Technology                                                               |
| ------------ | ------------------------------------------------------------------------ |
| UI Framework | React 16/17/18/19 (class components, no hooks)                           |
| Rendering    | Pixi.js 5/6 (WebGL), D3.js (zoom/scales/brushing)                        |
| Build        | Vite 5, TypeScript 5.7, Babel 7 (ES5 class compat)                       |
| Testing      | Vitest 3 + Playwright browser, MSW for API mocking                       |
| Linting      | Biome 1.9 (single quotes, semicolons, 80-char lines)                     |
| Styling      | SCSS modules, CSS injected via JS                                        |
| State Mgmt   | Custom Pub/Sub (`pub-sub-es`) + React component state (no Redux/Context) |

## Source Code Structure (`app/scripts/`)

```
app/scripts/
├── hglib.jsx                  # Library entry point
├── HiGlassComponent.jsx       # Root React component (~166KB)
├── TiledPlot.jsx              # View container (~91KB)
├── TrackRenderer.jsx          # Track rendering engine (~79KB)
├── api.js                     # Public API (~36KB)
├── configs/                   # Track types, colormaps, defaults
├── data-fetchers/             # Data source implementations
├── services/                  # Tile proxy, worker, chrom-info, DOM events
├── utils/                     # 100+ utility modules (incl. react-dom-compat.js)
├── hocs/                      # HOCs: pub-sub, theme, modal
├── plugins/                   # Plugin system for custom tracks
└── [150+ track implementations]
```

## Track Class Hierarchy

```
Track (base — app/scripts/Track.js)
  └─ PixiTrack (Pixi.js graphics, labels, SVG export — app/scripts/PixiTrack.js)
       └─ TiledPixiTrack (tile fetching/caching, zoom levels — app/scripts/TiledPixiTrack.js)
            ├─ Tiled1DPixiTrack → BarTrack, LineTrack, BedLikeTrack...
            └─ HeatmapTiledPixiTrack (2D heatmaps)
```

30+ track types: heatmaps, gene annotations, line/bar charts, arrowhead domains, CNV intervals, map tiles, and more.

## Key NPM Scripts

| Command              | Purpose                                        |
| -------------------- | ---------------------------------------------- |
| `npm start`          | Vite dev server (port 5173)                    |
| `npm run build`      | TypeScript emit + Vite build (UMD + ESM)       |
| `npm run typecheck`  | `tsc` type checking                            |
| `npm run lint`       | Biome CI linting                               |
| `npm run format`     | Biome auto-format                              |
| `npm test`           | Vitest (browser mode, 4 parallel shards in CI) |
| `npm run test-watch` | Vitest watch with visible browser              |

## CI/CD (`.github/workflows/`)

- **ci.yml**: Typecheck → Lint → Schema validation → Tests (4 shards, Playwright)
- **npmpublish.yml**: Publishes to npm on `v*` tags
- **docs.yml**: Builds Sphinx docs, deploys to S3

## Contributing Guidelines

- PRs require a **changelog entry** in `CHANGELOG.md`
- Use `[WIP]` prefix for in-progress PRs
- **Biome** enforces coding style — run `npm run format` before committing
- Docs use **Sphinx** with reStructuredText

## Architecture Highlights

- **No global state store** — uses Pub/Sub events + React class component state + instance variables
- **Plugin system** — register custom tracks via `window.higlassTracksByType` and data fetchers via `window.higlassDataFetchersByType`
- **Data flow**: Track → DataFetcher → tile-proxy (batched/debounced) → HTTP to server → Web Worker processing → LRU cache → Track render
- **Dual output**: UMD (`dist/hglib.js`) and ESM (`dist/higlass.mjs`), with React & Pixi.js as peer dependencies
- **SVG export** support for publication-quality figures

## React Multi-Version Support

HiGlass supports React 16, 17, 18, and 19 as peer dependencies. Key details:

- **Rendering compat layer** (`app/scripts/utils/react-dom-compat.js`): Detects `createRoot` (React 18+) at runtime and falls back to legacy `ReactDOM.render` (React 16/17). All mounting/unmounting goes through `renderToContainer`/`unmountFromContainer`.
- **`ref` callback pattern**: Class component refs use callback refs (`ref={(r) => { ... }}`), which work across all React versions. React 19 passes `ref` as a regular prop to function components, so function components that receive `ref` must not rely on `forwardRef`.
- **`hglib.launch()`** (`app/scripts/hglib.jsx`): After rendering, polls for `trackRenderer` readiness on each view because React 18/19 `createRoot` renders asynchronously (unlike `ReactDOM.render`).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [higlass/higlass](https://github.com/higlass/higlass) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
