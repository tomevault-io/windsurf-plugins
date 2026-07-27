---
trigger: always_on
description: Guidance for AI agents working in this repository.
---

# AGENTS.md (react-flow-smart-edge)

Guidance for AI agents working in this repository.

## What this project is

`@tisoap/react-flow-smart-edge` is a published npm library (MIT) that provides custom [React Flow](https://reactflow.dev) edges which route around nodes using grid-based A\* pathfinding.

- Consumers are React apps using `@xyflow/react` v12+ (peer dependency).
- This repo contains library source, Vite library build, Docusaurus docs site, Storybook demos/tests, and browser-based Storybook tests.
- Documentation: https://tisoap.github.io/react-flow-smart-edge/docs (Docusaurus, deployed to gh-pages on release)
- Storybook demos: Chromatic (`.github/workflows/chromatic.yml`); local dev via `npm run storybook`
- Package entry: `src/index.tsx` → `dist/index.{mjs,cjs}` + `dist/index.d.ts`

Do not treat Storybook stories or `src/demos/` as part of the public API unless explicitly exporting them.

## Architecture (read this before changing path logic)

Smart edges follow a fixed pipeline. Changes usually touch one stage:

```
nodes + edge endpoints
  → getBoundingBoxes()     // graph + per-node boxes (nodePadding, gridRatio)
  → createGrid()           // 2D walkability grid; mark node cells blocked
  → guaranteeWalkablePath() // ensure start/end cells are reachable
  → generatePath()         // A* on grid (diagonal or orthogonal)
  → gridToGraphPoint()     // grid coords → flow graph coords
  → drawEdge()             // point sequence → SVG `d` string
```

| Layer            | Location                                          | Role                                                                                         |
| ---------------- | ------------------------------------------------- | -------------------------------------------------------------------------------------------- |
| React components | `src/Smart*Edge/`, `src/SmartEdge/`               | Wire `useNodes()` + `getSmartEdge()` into `@xyflow/react` `BaseEdge`                         |
| Edge factory     | `src/createSmartEdge/`, `src/smartEdgePresets.ts` | `createSmartEdge(preset, options?)` and canonical preset configs                             |
| Core API         | `src/getSmartEdge/index.ts`                       | Pure(ish) path computation; returns `{ svgPathString, edgeCenterX, edgeCenterY }` or `Error` |
| Geometry / grid  | `src/functions/`                                  | Bounding boxes, grid creation, coordinate conversion, SVG drawing                            |
| Pathfinding      | `src/pathfinding/`                                | Grid type + A\* (based on [PathFinding.js](https://github.com/qiao/PathFinding.js))          |

### Edge presets (canonical configs in `src/smartEdgePresets.ts`)

| Export / preset         | `drawEdge`                    | `generatePath`                   | Fallback (on failure) |
| ----------------------- | ----------------------------- | -------------------------------- | --------------------- |
| `SmartBezierEdge`       | `svgDrawSmoothLinePath`       | `pathfindingAStarDiagonal`       | `BezierEdge`          |
| `SmartStraightEdge`     | `svgDrawStraightLinePath`     | `pathfindingAStarDiagonal`       | `StraightEdge`        |
| `SmartStepEdge`         | `svgDrawStraightLinePath`     | `pathfindingJumpPointNoDiagonal` | `StepEdge`            |
| `SmartSmoothStepEdge`   | `svgDrawSmoothStepLinePath`   | `pathfindingJumpPointNoDiagonal` | `SmoothStepEdge`      |
| `SmartSimpleBezierEdge` | `svgDrawSimpleBezierLinePath` | `pathfindingAStarDiagonal`       | `SimpleBezierEdge`    |

Preset components are `createSmartEdge(preset)` with default options. Consumers can call `createSmartEdge("step", { gridRatio: 5 })` at module scope, or use exported `SmartEdge` + `smartEdgePresets` for custom rendering.

Custom edges that bypass presets should call `getSmartEdge({ ...edgeProps, nodes, options })` and handle `instanceof Error` (see README).

### Tunable options (`GetSmartEdgeOptions`)

- `nodePadding` (default `10`, min `2`): clearance around nodes in px.
- `gridRatio` (default `10`, min `2`): px per grid cell; lower = more accurate, slower.
- `drawEdge` / `generatePath`: pluggable; types exported from `src/index.tsx`.

## Repository layout

```
src/
  index.tsx              # Public exports only
  getSmartEdge/          # Core algorithm entry
  createSmartEdge/       # createSmartEdge factory
  smartEdgePresets.ts    # Canonical preset drawEdge/generatePath/fallback configs
  SmartEdge/             # Shared React wrapper (BaseEdge + fallback)
  SmartBezierEdge/       # Preset components (thin createSmartEdge wrappers)
  SmartStraightEdge/
  SmartStepEdge/
  SmartSmoothStepEdge/
  SmartSimpleBezierEdge/
  functions/             # Grid, bounds, SVG path builders
  pathfinding/           # Grid + A*
  demos/                 # Shared GraphWrapper, fixtures, demoRegistry (Storybook + docs)
  stories/               # Storybook only (excluded from dts build)
website/                 # Docusaurus documentation site (npm workspace)
.storybook/              # Storybook + Vitest browser setup
dist/                    # Build output (gitignored in dev; published to npm)
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tisoap/react-flow-smart-edge](https://github.com/tisoap/react-flow-smart-edge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
