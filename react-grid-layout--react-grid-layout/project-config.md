---
trigger: always_on
description: transformStrategy,
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

React-Grid-Layout is a draggable and resizable grid layout system for React with responsive breakpoints. It's a pure React implementation (no jQuery) used in production by BitMEX, Grafana, Metabase, HubSpot, and many others.

**Version 2** is a complete TypeScript rewrite with a modern hooks-based API while maintaining backwards compatibility through a dedicated legacy wrapper.

## Package Manager

**Always use `yarn`** instead of `npm` for all commands in this project.

## Development Commands

### Testing

```bash
# Run all tests with coverage
make test
yarn test

# Watch mode for development
make test-watch

# Run specific test file
NODE_ENV=test npx jest --testPathPatterns="compactors"
```

### Building

```bash
# Build the library (ESM, CJS, and TypeScript declarations)
make build
yarn build

# Clean build artifacts
make clean
```

### Development Server

```bash
# Start development server with hot reload (port 4002)
make dev
yarn dev
```

### Linting & Formatting

```bash
# Run ESLint
yarn lint

# Format code with Prettier (run before committing!)
yarn fmt
```

**Important**: Always run `yarn fmt` before committing. The CI will fail if code is not formatted.

### Releasing

```bash
# Patch release (bug fixes) - bumps version, builds, publishes to npm
make release-patch

# Minor release (new features, backwards compatible)
make release-minor

# Major release (breaking changes)
make release-major
```

**Important**: Always use `make release-*` commands, never `npm version` directly. The Makefile handles building, version bumping, and publishing correctly.

## Architecture (v2)

### Package Structure

```
src/
├── core/                    # Pure TypeScript, no React dependencies
│   ├── types.ts             # All type definitions
│   ├── layout.ts            # Layout manipulation (move, clone, validate)
│   ├── collision.ts         # Collision detection
│   ├── sort.ts              # Sorting algorithms
│   ├── compactors.ts        # Compaction algorithms (vertical, horizontal)
│   ├── compact-compat.ts    # Legacy compact() function wrapper
│   ├── constraints.ts       # Layout constraints (position, size, aspect ratio)
│   ├── calculate.ts         # Grid calculations (grid units <-> pixels)
│   ├── position.ts          # CSS positioning helpers
│   ├── responsive.ts        # Breakpoint utilities
│   └── index.ts             # Core exports
│
├── react/                   # React bindings
│   ├── hooks/
│   │   ├── useContainerWidth.ts   # Container width measurement
│   │   ├── useGridLayout.ts       # Grid state management
│   │   └── useResponsiveLayout.ts # Responsive breakpoint handling
│   └── components/
│       ├── GridItem.tsx           # Individual grid item
│       ├── GridLayout.tsx         # Main grid component
│       ├── ResponsiveGridLayout.tsx
│       └── WidthProvider.tsx      # Width measurement HOC (internal)
│
├── legacy/                  # v1 API compatibility
│   ├── ReactGridLayout.tsx        # Legacy component wrapper
│   ├── ResponsiveReactGridLayout.tsx
│   ├── WidthProvider.tsx          # Re-exports for backwards compat
│   └── index.ts
│
└── index.ts                 # Main entry point
```

### Entry Points

```typescript
// New v2 API (recommended)
import ReactGridLayout, {
  Responsive,
  useContainerWidth,
  verticalCompactor,
  horizontalCompactor
} from "react-grid-layout";

// With composable interfaces
<ReactGridLayout
  width={width}
  layout={layout}
  gridConfig={{ cols: 12, rowHeight: 30 }}
  dragConfig={{ enabled: true, handle: '.handle' }}
  resizeConfig={{ enabled: true, handles: ['se'] }}
  compactor={verticalCompactor}
/>

// Core utilities (framework-agnostic)
import {
  moveElement,
  collides,
  transformStrategy,
  absoluteStrategy,
  createScaledStrategy,
  getCompactor,
  verticalCompactor,
  horizontalCompactor
} from "react-grid-layout/core";
// Note: compact() is not exported. Use compactor.compact() instead.

// Legacy v1 API (100% backwards compatible, flat props)
import ReactGridLayout, {
  WidthProvider,
  Responsive
} from "react-grid-layout/legacy";
```

### Core Components

**GridLayout** (`src/react/components/GridLayout.tsx`)

- Main grid layout component (functional, hooks-based)
- Manages layout state, drag/drop, and resize operations
- Handles compaction (vertical, horizontal, or none)
- All grid items must have a unique `key` prop matching `i` in layout

**ResponsiveGridLayout** (`src/react/components/ResponsiveGridLayout.tsx`)

- Wraps GridLayout with responsive breakpoint support
- Manages multiple layouts keyed by breakpoint
- Automatically generates missing breakpoint layouts

**GridItem** (`src/react/components/GridItem.tsx`)

- Individual grid item wrapper
- Integrates with react-draggable and react-resizable
- Handles positioning via CSS transforms (default)

### Core Algorithms

**Compaction** (`src/core/compactors.ts`)

- `verticalCompactor`: Items float up (default)
- `horizontalCompactor`: Items float left
- `noCompactor`: Free positioning
- All implement the `Compactor` interface

**Collision Detection** (`src/core/collision.ts`)

- `collides()`: Check if two items overlap
- `getFirstCollision()`: Find first collision

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [react-grid-layout/react-grid-layout](https://github.com/react-grid-layout/react-grid-layout) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
