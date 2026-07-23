---
trigger: always_on
description: This document defines the coding practices, architecture, and conventions for the Ripl project. It is intended to provide AI coding agents with the context needed to contribute effectively.
---

# AGENTS.md — Ripl Coding Agent Guide

This document defines the coding practices, architecture, and conventions for the Ripl project. It is intended to provide AI coding agents with the context needed to contribute effectively.

## Project Overview

Ripl (pronounced "ripple") is a library that provides a **unified API for 2D graphics rendering** (Canvas & SVG) in the browser, with a focus on high performance and interactive data visualization. Key design goals:

- **Unified API** — One API surface for both Canvas and SVG rendering; switching contexts is a single-line change
- **DOM/CSSOM mimicry** — Grouping, property inheritance, event bubbling/delegation, CSS-like querying (`query`, `queryAll`, `getElementById`, etc.)
- **Zero runtime dependencies** — All functionality is self-contained
- **Tree-shakable** — Fully modular; consumers only ship what they use
- **Strict TypeScript** — The entire codebase is strictly typed

## Communication Style
In all interactions and commit messages, be extremely concise and sacrifice grammar for the sake of concision.

## Repository Structure

Yarn 4 monorepo with workspaces:

```
packages/
├── core/         # Core rendering: elements, scene, renderer, animation, scales, math, color, interpolation
├── canvas/       # Canvas 2D rendering context
├── svg/          # SVG context implementation
├── charts/       # Pre-built chart components (bar, line, area, pie, radar, heatmap, etc.)
├── 3d/           # 3D rendering (experimental)
├── webgpu/       # WebGPU 3D rendering context
├── terminal/     # Terminal rendering context
├── node/         # Node.js runtime bindings
├── web/          # Main browser entry point
├── dom/          # DOM utilities
├── utilities/    # Shared typed utility functions
└── test-utils/   # Test utilities
app/              # Documentation site (VitePress) with demos
```

Each package follows:
```
src/          # Source code
test/         # Tests (mirrors src/ structure)
package.json
tsconfig.json
```

## Architecture & Key Abstractions

### Class Hierarchy

```
EventBus<TEventMap>
├── Element<TState, TEventMap>     # Base renderable with state, events, interpolation
│   ├── Shape<TState>              # Adds path management, autoFill/autoStroke, hit testing
│   │   ├── Circle, Rect, Arc, Line, Polygon, Polyline, Ellipse, Text, Path, Image
│   │   └── (custom shapes)
│   └── Group<TEventMap>           # Container with children, scenegraph, querying
│       └── Scene                  # Top-level group bound to a Context
├── Renderer                       # Animation loop with transition management
└── Chart<TOptions, TEventMap>     # Base chart (in @ripl/charts)
```

### Context Abstraction

`Context` is the rendering abstraction. `@ripl/web` is the main entry point for browser usage — it re-exports everything from `@ripl/core` (elements, scene, renderer, animation, scales, etc.) and `@ripl/canvas` (Canvas context), and sets up browser-specific platform bindings. For SVG rendering, import `createContext` from `@ripl/svg` instead. All elements render through `Context`, making them context-agnostic.

### Scenegraph

Elements are organized in `Group` trees. `Scene` maintains a **hoisted buffer** — a flattened list of all renderable elements — converting render traversal from O(n^c) to O(n). The `Renderer` drives the animation loop via `requestAnimationFrame`.

### Event System

`EventBus` provides `on`, `off`, `once`, `emit` with:
- **Typed events** via `EventMap` generics
- **Event bubbling** through parent chain (mimics DOM capture phase)
- **Stop propagation** via `event.stopPropagation()`
- **Disposable subscriptions** — `on()` returns `{ dispose }` for cleanup
- **Self-only filtering** via `{ self: true }` subscription option

## Element & Shape Patterns

### Defining a Built-in Element

Every element follows this exact pattern:

1. **State interface** extending `BaseElementState`
2. **Class** extending `Shape<TState>` with getter/setter pairs using `getStateValue`/`setStateValue`
3. **Constructor** calling `super(type, options)` with a string type name
4. **`_getLocalBoundingBox()`** returning the element's raw (untransformed) local-space `Box` — the base `Element` composes the public `getBoundingBox(local?)` (world box by default, local when `local` is `true`) from it
5. **`render(context)`** calling `super.render(context, path => { ... })`
6. **Factory function** `createX()`
7. **Type guard** `elementIsX()`

Example (from `circle.ts`):

```typescript
export interface CircleState extends BaseElementState {
    cx: number;
    cy: number;
    radius: number;
}

export class Circle extends Shape<CircleState> {

    public get cx() {
        return this.getStateValue('cx');
    }

    public set cx(value) {
        this.setStateValue('cx', value);
    }

    // ... cy, radius follow the same pattern

    constructor(options: ShapeOptions<CircleState>) {
        super('circle', options);
    }

    public _getLocalBoundingBox(): Box {
        return new Box(
            this.cy - this.radius,
            this.cx - this.radius,
            this.cy + this.radius,
            this.cx + this.radius
        );
    }


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [andrewcourtice/ripl](https://github.com/andrewcourtice/ripl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
