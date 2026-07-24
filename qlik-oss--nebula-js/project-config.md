---
trigger: always_on
description: Nebula.js is a framework for building and embedding Qlik visualizations. It's a **Lerna monorepo** managing multiple packages under `@nebula.js` npm scope. The core architecture separates visualization authoring (`supernova`) from rendering (`nucleus`) with `stardust` as the public API facade.
---

# Nebula.js Development Guide for AI Agents

## Project Overview

Nebula.js is a framework for building and embedding Qlik visualizations. It's a **Lerna monorepo** managing multiple packages under `@nebula.js` npm scope. The core architecture separates visualization authoring (`supernova`) from rendering (`nucleus`) with `stardust` as the public API facade.

## Architecture

### Package Structure

- **`apis/stardust/`** - Public API that exposes `nucleus` and `supernova` APIs. Entry point for consumers.
- **`apis/nucleus/`** - React-based mashup runtime that renders visualizations in web apps
- **`apis/supernova/`** - Hooks-based API for building custom visualizations (the "component" layer)
- **`apis/locale/`** - Translation strings and locale generation (private)
- **`apis/theme/`** - Theme access and consumption (private)
- **`apis/conversion/`** - Conversion utilities for hyperCube extensions (private)
- **`apis/enigma-mocker/`** - Mock enigma app for rendering without Qlik engine
- **`apis/snapshooter/`** - Chart rendering as images
- **`apis/test-utils/`** - Testing utilities
- **`commands/`** - CLI tools (`build`, `serve`, `create`, `sense`) for visualization development
- **`packages/ui/`** - Shared UI components (private)

### Visualization Component Pattern (Supernova)

Visualizations are defined as "supernovas" with this structure:

```js
export default function supernova(galaxy) {
  return {
    qae: {
      properties: {},  // QIX object properties schema
      data: {}        // Data definition (dimensions, measures)
    },
    component() {
      // Hooks-based component using @nebula.js/stardust hooks
      const element = useElement();
      const layout = useLayout();
      
      useEffect(() => {
        // Render logic here
      }, [layout]);
    }
  };
}
```

Key hooks from `@nebula.js/stardust`:
- `useElement()` - Get the DOM element
- `useLayout()` - Get QIX layout object  
- `useModel()` - Access QIX model
- `useStaleLayout()` - Layout that doesn't trigger re-renders
- `useSelections()` - Handle user selections
- `useConstraints()` - Interaction constraints (edit, select, active, passive)
- `useEffect()`, `useState()`, etc. - React-like hooks

See [apis/supernova/src/hooks.js](apis/supernova/src/hooks.js) for complete hook implementations.

### Nucleus Rendering Flow

Nucleus (React-based) orchestrates visualization rendering:
1. `Supernova.jsx` component mounts the visualization element
2. Calls supernova lifecycle: `created()` → `mounted()` → `render()` → `willUnmount()`
3. Passes context (constraints, interactions, theme, appLayout) to component
4. Uses `RenderDebouncer` to batch updates

## Development Workflow

### Setup & Build

```bash
yarn install                    # Install all dependencies
yarn run locale:generate        # Generate translation files (required before build)
yarn run build                  # Build all packages (production UMD + ESM)
yarn run build:dev              # Build with source maps for development
yarn run build:watch            # Watch mode for active development
```

**Critical**: Always run `locale:generate` before building if locale files changed.

### Testing

```bash
yarn test:unit                  # Jest unit tests for all packages
yarn test:component             # Playwright component tests
yarn test:integration           # Playwright integration tests  
yarn test:rendering             # Visual regression tests (Playwright)
yarn test:mashup                # Full mashup integration tests
```

Unit tests use `jest` with `jest-environment-jsdom`. See [jest.config.js](jest.config.js) for coverage and test patterns.

### CLI Commands for Visualizations

When developing a supernova visualization:

```bash
nebula create <name>            # Scaffold new visualization
nebula serve                    # Start dev server with hot reload
nebula build                    # Build visualization for distribution
nebula sense                    # Build as Qlik Sense extension
```

**Dev Server** (`nebula serve`):
- Builds visualization to `/dist` automatically
- Connects to Qlik Associative Engine (default: localhost:9076)
- Supports fixtures for testing without engine connection
- See [commands/serve/README.md](commands/serve/README.md) for options

### Rollup Build Configuration

All packages use shared [rollup.config.js](rollup.config.js):
- **UMD format** for browsers (default export via `unpkg`/`jsdelivr`)
- **ESM format** for bundlers (exposed via `module` field)
- Creates `.dev.js` variants with source maps in development mode
- Validates `main`, `module`, `unpkg`, `jsdelivr` targets in package.json match naming convention

### Lerna Versioning

This project uses Lerna for version management with **conventional commits**:
- Use commit types: `feat:`, `fix:`, `chore:`, `docs:`, etc.
- `lerna version` reads commits to determine semver bump
- All packages are versioned together (see [lerna.json](lerna.json))

## Code Conventions

### ESLint Configuration

- Uses **Airbnb style** with Prettier formatting

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [qlik-oss/nebula.js](https://github.com/qlik-oss/nebula.js) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
