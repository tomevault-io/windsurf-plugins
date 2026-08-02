---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Reagraph is a high-performance WebGL network graph visualization library for React (v4.30.7+). It provides interactive graph rendering using Three.js and React Three Fiber with support for 2D/3D layouts, node clustering, edge bundling, and various interaction modes.

**Repository:** https://github.com/reaviz/reagraph
**License:** Apache-2.0

## Development Commands

### Core Development
- `npm start` - Start Storybook development server on port 9009
- `npm run build` - Build library (`vite build --mode library`)
- `npm test` - Run tests with Vitest
- `npm run test:coverage` - Run tests with coverage

### Code Quality
- `npm run lint` - Run ESLint on TypeScript files
- `npm run lint:fix` - Fix ESLint issues automatically
- `npm run prettier` - Format code with Prettier

### Documentation
- `npm run build-storybook` - Build Storybook for production

### Pre-commit
- Uses Husky for git hooks with lint-staged
- ESLint and Prettier run automatically on staged files

## Architecture

### Core Components
- **GraphCanvas** (`src/GraphCanvas/`) - Main component wrapping React Three Fiber Canvas, entry point for the library
- **GraphScene** (`src/GraphScene.tsx`) - Core 3D scene management and rendering logic
- **Layout System** (`src/layout/`) - Multiple graph layout algorithms
- **Symbol System** (`src/symbols/`) - Renderable graph elements (nodes, edges, clusters, labels)
- **Store** (`src/store.ts`) - Zustand-based state management for graph interactions

### Layout Types (src/layout/types.ts)
Available layout algorithms:
- **Force-directed:** `forceDirected2d`, `forceDirected3d`
- **Tree layouts:** `treeTd2d`, `treeTd3d`, `treeLr2d`, `treeLr3d`
- **Radial:** `radialOut2d`, `radialOut3d`
- **Circular:** `circular2d`, `concentric2d`, `concentric3d`
- **Hierarchical:** `hierarchicalTd`, `hierarchicalLr`
- **Specialized:** `nooverlap`, `forceatlas2`, `custom`

### Node Sizing Types (src/sizing/)
- `default` - Fixed size for all nodes
- `attribute` - Size based on node data attribute
- `centrality` - Size based on graph centrality metrics
- `pageRank` - Size based on PageRank algorithm

### Symbol Components (src/symbols/)
- **Nodes:** `Sphere`, `Icon`, `Svg`, `SphereWithIcon`, `SphereWithSvg`, `Badge`
- **Edges:** `Edge`, `Edges`, `SelfLoop` with animation and geometry hooks
- **Clusters:** `Ring` for cluster visualization
- **Labels:** `Label` for node/edge text

### Key Systems
- **Layout Providers** (`src/layout/layoutProvider.ts`) - Factory for layout algorithms
- **Node Sizing** (`src/sizing/nodeSizeProvider.ts`) - Dynamic node sizing strategies
- **Camera Controls** (`src/CameraControls/`) - 3D camera interaction (pan, orbit, orthographic)
- **Selection System** (`src/selection/`) - Node selection with lasso and click interactions
- **Collapse System** (`src/collapse/`) - Node collapsing/expanding functionality
- **Theming** (`src/themes/`) - Light/dark themes and customizable styling
- **Radial Menu** (`src/RadialMenu/`) - Context menu for node interactions

### Data Flow
1. Graph data (`nodes`/`edges`) passed to `GraphCanvas`
2. `useGraph` hook builds graphology graph and runs layout algorithms
3. Layout positions stored in Zustand store
4. Positioned graph elements rendered as Three.js objects via React Three Fiber
5. User interactions handled via R3F event system
6. State updates trigger re-renders through store subscriptions

### Key Hooks
- `useGraph` - Main hook managing graph state, layout, and transformations
- `useStore` - Zustand store access with shallow comparison
- `useCameraControls` / `useCenterGraph` - Camera manipulation
- `useSelection` - Selection state management
- `useCollapse` - Node collapse/expand logic
- `useDrag` - Node dragging with gesture handling
- `useHoverIntent` - Hover detection with intent patterns
- `useEdgeEvents` / `useEdgeAnimations` / `useEdgeGeometry` - Edge-specific hooks

### Testing
- Uses Vitest with jsdom environment
- Test files use `.test.ts` suffix
- Coverage reports available via `npm run test:coverage`
- Tests focus on utilities and business logic (not React components)

### Build System
- Vite with TypeScript (`vite.config.mts`)
- Dual build modes: `library` (for publishing) and `development` (for Storybook)
- CSS modules for component styles (`.module.css`)
- SVG imports via `vite-plugin-svgr`
- Type declarations via `vite-plugin-dts`
- CSS injected by JS via `vite-plugin-css-injected-by-js`

### Storybook
- Stories in `stories/demos/*.story.tsx`
- Uses `@storybook/react-vite` framework
- Configured in `.storybook/main.ts`

## Coding Rules and Conventions

### TypeScript Patterns
- All component props interfaces end with `Props` suffix (e.g., `NodeProps`, `EdgeProps`, `GraphCanvasProps`)
- Use `FC<Props>` type for functional components
- Destructure props with default values in parameter list: `draggable = false`
- Export types and interfaces alongside components for external consumption
- Use `InternalGraph*` prefix for internal data structures (e.g., `InternalGraphNode`, `InternalGraphEdge`)
- Type imports use `import type` syntax

### Component Structure

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [reaviz/reagraph](https://github.com/reaviz/reagraph) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
