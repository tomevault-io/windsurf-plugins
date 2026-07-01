---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Dockview is a zero-dependency layout manager supporting tabs, groups, grids and splitviews. It provides framework support for React, Vue, Angular, and JavaScript. The project is organized as an NX monorepo (package-based approach) with Yarn v1 workspaces.

See per-package `AGENTS.md` files under `packages/` for package-specific guidance.

## Development Commands

### Build

-   `yarn build` - Build all publishable packages via NX (dockview-core, dockview, dockview-vue, dockview-react, dockview-angular)
-   `yarn clean` - Clean all packages
-   `npx nx run <package>:<script>` - Run a specific script for a single package (e.g. `npx nx run dockview-core:build`)

### Build Order

NX handles build ordering automatically via `dependsOn: ["^build"]`. The dependency chain is:

    dockview-core → dockview → dockview-react
    dockview-core → dockview → dockview-vue
    dockview-core → dockview → dockview-angular

### CSS Flow

-   `dockview-core` compiles SCSS to CSS via Gulp (`gulp sass`)
-   All other packages copy CSS from `dockview-core` using `scripts/copy-css.js`

### Testing

-   `yarn test` - Run Jest tests across all packages via NX
-   `yarn test:cov` - Run tests with coverage (root-level Jest invocation for SonarCloud unified coverage)

### Linting & Formatting

-   `yarn lint` - Run ESLint across all packages via NX
-   `yarn lint:fix` - Run ESLint with automatic fixing
-   `yarn format` - Run Prettier across all packages
-   `yarn format:check` - Check Prettier formatting

### Documentation

-   `yarn docs` - Generate TypeDoc documentation

### Release

-   `yarn release` - NX release (fixed versioning, all packages share same version)
-   `yarn release:version` - Bump version
-   `yarn release:publish` - Publish to npm

## Architecture

### Monorepo Structure

-   **packages/dockview-core** - Internal core layout engine (TypeScript, framework-agnostic, zero dependencies). Not a documented install target — consumers use `dockview` or a framework binding.
-   **packages/dockview** - Canonical JavaScript package (`npm install dockview`). Batteries-included: re-exports the core API and registers the separable feature modules. No `react` peer dependency.
-   **packages/dockview-react** - Canonical React bindings package — what docs, READMEs, and examples point users at (`npm install dockview-react`). Holds the actual React source and depends on `dockview`.
-   **packages/dockview-vue** - Vue 3 bindings and components
-   **packages/dockview-angular** - Angular bindings and components
-   **packages/docs** - Documentation website (Docusaurus v3)

### Key Components

#### Core Architecture (dockview-core)

-   **DockviewComponent** - Main container managing panels and groups
-   **DockviewGroupPanel** - Container for related panels with tabs
-   **DockviewPanel** - Individual content panels
-   **Gridview/Splitview/Paneview** - Different layout strategies
-   **API Layer** - Programmatic interfaces for each component type

#### Framework Integration

-   Framework-specific packages provide thin wrappers around core components
-   React package uses HOCs and hooks for component lifecycle management
-   Vue package provides Vue 3 composition API integration
-   All frameworks share the same core serialization/deserialization logic

#### Key Features

-   Drag and drop with customizable drop zones
-   Floating groups and popout windows
-   Edge groups (pinned to layout edges with collapse/expand)
-   Tab groups (colored chip-based visual tab organization)
-   Serialization/deserialization for state persistence
-   Theming system with CSS custom properties
-   Comprehensive API for programmatic control

### Build System

-   **NX** for monorepo orchestration (package-based, `useInferencePlugins: false`)
-   **Yarn v1** for package management and workspaces
-   **TypeScript** (`tsc`) for CJS + ESM compilation
-   **Gulp** for SCSS processing (dockview-core only)
-   **Rollup** for UMD bundles (dockview-core, dockview, dockview-react)
-   **Vite** for Vue package builds
-   **ng-packagr** for Angular Package Format builds

### Testing Strategy

-   Jest with ts-jest preset for TypeScript support
-   Testing Library for React component testing
-   Coverage reporting with SonarCloud integration
-   Each package has its own jest.config.ts extending root configuration

### Code Quality

-   ESLint configuration extends recommended TypeScript rules
-   Prettier for code formatting
-   Linting targets source files in packages/\*/src/\*\* (excludes tests, docs, node_modules)
-   Current rules focus on TypeScript best practices while allowing some flexibility

## Development Notes

### Working with Packages

-   Use NX commands for cross-package operations (`npx nx run-many -t <target>`)
-   Each package can be built independently via `npx nx run <package>:build`
-   Core package must be built before framework packages (NX handles this automatically)

### Adding New Features

-   Start with core package implementation
-   Add corresponding API methods in api/ directory
-   Create framework-specific wrappers as needed

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mathuo/dockview](https://github.com/mathuo/dockview) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
