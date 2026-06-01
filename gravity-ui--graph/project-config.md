---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

@gravity-ui/graph is a graph visualization library that combines Canvas for high-performance rendering with HTML/React for rich interactions. The library automatically switches between rendering modes based on zoom level.

## Development Commands

```bash
# Install dependencies (use npm, not yarn or pnpm)
npm install

# Development mode (watch TypeScript and CSS)
npm run dev

# Run Storybook for development
npm run storybook

# Build for production
npm run build:publish

# Type checking
npm run typecheck

# Linting
npm run lint

# Testing
npm run test

# Update snapshots
npm run test -- --updateSnapshot

# Build Storybook
npm run build-storybook
```

## Core Architecture

### Dual Rendering System

The library uses a **hybrid Canvas + React architecture**:

- **Canvas Mode (Zoomed Out)**: Entire graph rendered on Canvas for maximum performance with thousands of elements
- **React Mode (Zoomed In)**: React components activated for blocks when camera scale reaches threshold
- **Automatic Switching**: `ReactLayer` manages the transition based on `activationScale` configuration

**Key Files**:
- `src/components/canvas/layers/graphLayer/GraphLayer.ts` - Main Canvas rendering
- `src/react-components/layer/ReactLayer.tsx` - React Portal integration
- `src/react-components/GraphCanvas.tsx` - React wrapper component

### Custom Component Framework

The library implements a **custom component system** (not React) for Canvas rendering:

**Component Hierarchy**:
1. `CoreComponent` (`src/lib/CoreComponent.ts`) - Tree structure, children management, context propagation
2. `Component` (`src/lib/Component.ts`) - Lifecycle hooks, state/props management
3. `GraphComponent` (`src/components/canvas/GraphComponent/`) - HitBox, dragging, ports for graph elements
4. `Block`, `BlockConnection` - Specific implementations

**Lifecycle Flow**:
```
willMount → firstIterate → willRender → render → didRender
         → willUpdateChildren → didUpdateChildren
         → propsChanged/stateChanged/contextChanged
         → unmount
```

**Update Pattern**: Components call `performRender()` to schedule updates via the scheduler, which batches renders in the next animation frame.

### Layer System

Layers are the primary extension mechanism. Each layer is a Component that manages Canvas and/or HTML rendering:

**Built-in Layers (render order)**:
1. **BelowLayer** (zIndex: 1) - Background grid
2. **GraphLayer** (zIndex: 2) - Blocks and connections (Canvas)
3. **SelectionLayer** (zIndex: 3) - Selection visualization
4. **ReactLayer** (zIndex: 3) - React components (HTML)
5. **CursorLayer** (zIndex: 4+) - Dynamic cursor management

**Layer Lifecycle**:
- `init()` - Create Canvas/HTML elements
- `attachLayer()` - Attach to DOM, call `afterInit()`
- `afterInit()` - **IMPORTANT**: Set up ALL event listeners here (NOT in constructor/init)
- `unmount()` - Clean up via AbortController

**Creating Custom Layers**:
```typescript
class MyLayer extends Layer {
  constructor(props) {
    super({
      canvas: { zIndex: 2, respectPixelRatio: true },
      html: { zIndex: 3, transformByCameraPosition: true },
      ...props
    });
  }

  protected afterInit() {
    // Subscribe to events using wrapper methods
    this.onGraphEvent("camera-change", this.handleCameraChange);
    this.onCanvasEvent("mousedown", this.handleMouseDown);
    super.afterInit(); // Call at end
  }
}

// Add to graph
graph.addLayer(MyLayer, { customProp: 'value' });
// Do NOT pass graph, camera, root - these are auto-provided
```

### Scheduler System

**GlobalScheduler** (`src/lib/Scheduler.ts`) drives all rendering via `requestAnimationFrame`:

- **Priority Queues**: 5 levels (HIGHEST → LOWEST)
- **Batched Updates**: `performRender()` marks components dirty, actual render happens in next frame
- **Tree Traversal**: Only dirty components actually call `render()`
- **Lifecycle**: `graph.start()` begins animation loop, `graph.stop()` pauses

### Reactive State with Preact Signals

The library uses `@preact/signals-core` for reactive state management:

**Store Structure** (`src/store/`):
```
RootStore
├── blocksList: BlockListStore
│   ├── $blocks: Signal<BlockState[]>
│   └── blockSelectionBucket
├── connectionsList: ConnectionsStore
│   ├── $connections: Signal<ConnectionState[]>
│   └── connectionSelectionBucket
├── groupsList: GroupsListStore
├── settings: GraphEditorSettings
└── selectionService: SelectionService
```

**Block State** (`src/store/block/Block.ts`):
```typescript
BlockState {
  $rawState: Signal<TBlock>           // Raw block data
  $state: computed(() => ...)         // Full state + selection
  $geometry: computed(() => ...)      // x, y, width, height
  $selected: computed(() => ...)      // Selection state
  $anchorStates: Signal<AnchorState[]>
}
```

**Port State** (`src/store/connection/port/Port.ts`):
```typescript
PortState {
  $state: Signal<TPort>              // Raw port data (id, x, y, component, lookup)
  $point: computed(() => ...)         // Effective position (respects delegation)
  delegate(target): void              // Mirror another port's position

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gravity-ui/graph](https://github.com/gravity-ui/graph) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->
