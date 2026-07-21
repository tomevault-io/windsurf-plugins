---
trigger: always_on
description: This file mirrors `CLAUDE.md` so Codex-style agents can read the same repository guidance.
---

# AGENTS.md

This file mirrors `CLAUDE.md` so Codex-style agents can read the same repository guidance.

## Project Overview

Flitter is a JavaScript rendering engine and framework inspired by Flutter, designed for creating high-performance graphics and user interfaces in web applications. It provides a declarative API with support for both SVG and Canvas rendering.

## Architecture

### Core Concepts
- **Rendering Engine**: Manages the render object tree with dual renderer support (SVG/Canvas)
- **Widget System**: Flutter-like widgets (StatefulWidget, StatelessWidget, RenderObjectWidget)
- **Layout System**: Constraint-based box model layout
- **Animation Framework**: Built-in animation controllers, curves, and tweens

### Monorepo Structure
```
packages/
├── flitter/          # Core rendering engine
├── flitter-react/    # React integration
├── flitter-svelte/   # Svelte integration
├── chart/            # Chart library (shadcn-style, CLI-installable)
├── docs/             # Documentation site (Astro)
├── story/            # Storybook components
└── test/             # Test suite
shared/
├── chart-presets/    # Generated chart presets (synced from templates, do NOT edit directly)
└── ...               # Other shared diagram components
dev/
├── chart-storybook/  # Chart visual review (Storybook at localhost:6007)
└── shared/chart.ts   # Re-exports chart-presets
todo/
└── todo.md           # Chart work guide and feedback checklist
```

## Chart Library

### Overview

The chart library (`packages/chart/`) is a **shadcn-style** chart system built on top of Flitter. Users install charts via a CLI that copies template source code into their project — not as a compiled dependency. This means templates must be self-contained, readable, and editable by end users.

### Two Style Directions

Every chart ships with two visual styles:
- **Ag** — mimics AG Charts (structural, restrained, opacity-driven hover)
- **Toast** — mimics TOAST UI Chart (lifted, contrasted, outline-driven hover)

### Architecture Layers

```
1. Headless Engine    packages/chart/src/headless/<chart>/
   Pure logic: scales, layout, data transforms. No visuals.

2. Templates          packages/chart/registry/templates/
   Source of truth for styled charts. This is what the CLI copies.

3. Presets (generated) shared/chart-presets/
   Auto-synced from templates. NEVER edit directly.

4. Storybook          dev/chart-storybook/
   Visual review surface. Consumes presets via dev/shared/chart.ts.
```

### Template Structure

```
registry/templates/
├── charts/<chart-name>/
│   ├── base/              # Chart-specific shared logic (both styles use this)
│   ├── styles/ag/         # AG style: config + parts
│   ├── styles/toast/      # Toast style: config + parts
│   ├── index.ts           # Chart entry point
│   └── plugin.ts          # CLI plugin metadata
├── shared/
│   ├── bar-like/          # Shared base for bar-shaped charts
│   ├── line-like/         # Shared base for line-shaped charts
│   └── point-like/        # Shared base for point-shaped charts
└── styles/
    ├── ag/                # AG global style parts (legend, title, tooltip, cartesian axes)
    └── toast/             # Toast global style parts (legend, title, tooltip, cartesian axes)
```

### Headless Engines (23 total)

Not every Storybook chart has its own headless engine. Some reuse another:
- `AreaChart` and `StackedAreaChart` use `line-chart` headless
- `StackedBarChart` uses `bar-chart` headless

### Preset Sync Workflow

After editing files under `packages/chart/registry/templates/`, sync to regenerate `shared/chart-presets/`:

```bash
pnpm --dir shared/chart-presets run sync -- --chart <chart-name>           # specific chart
pnpm --dir shared/chart-presets run sync -- --chart <name> --style <style> # specific style
pnpm --dir shared/chart-presets run sync -- --changed                      # git-changed only
pnpm --dir shared/chart-presets run sync                                   # full sync
```

## Development Commands

```bash
# Build core library
npm run flitter:build

# Documentation
npm run docs:start      # Start dev server
npm run docs:build      # Build site

# Storybook
npm run story:start     # Start Storybook
npm run story:build     # Build Storybook

# Testing
npm run test:dev        # Run tests in dev mode
npm run test:playwright # Run integration tests
```

## Code Patterns

### Widget Creation
Widgets are exported as factory functions:
```typescript
export default function Container(props: ContainerProps): Widget {
  return new _Container(props);
}
```

### Render Object Widgets
```typescript
class MyRenderWidget extends SingleChildRenderObjectWidget {
  createRenderObject(): RenderObject {
    return new MyRenderObject();
  }
}
```

### State Management
```typescript
class MyWidget extends StatefulWidget {
  createState(): State<MyWidget> {
    return new MyWidgetState();
  }
}
```

## Key Development Areas

- **Widget Development**: `packages/flitter/src/component/`
- **Render Objects**: `packages/flitter/src/renderobject/`
- **Animation System**: `packages/flitter/src/animation/`
- **Tests**: `packages/test/tests/`
- **Documentation**: `packages/docs/src/content/`

## Testing & Quality


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [meursyphus/flitter](https://github.com/meursyphus/flitter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
