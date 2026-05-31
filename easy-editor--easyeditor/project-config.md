---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

EasyEditor is a plugin-based cross-framework low-code engine for building visual application platforms. This is the core engine monorepo that provides the foundation for the entire ecosystem.

## Build & Development Commands

### Essential Commands
```bash
# Install dependencies (pnpm 9.12.2+, node >= 18.0.0)
pnpm install

# Build all @easy-editor/* packages
pnpm build

# Generate TypeScript declarations for all packages
pnpm types

# Combined build + types (run before publishing)
pnpm pub:build

# Run dashboard example
pnpm example:dashboard

# Development mode (all packages)
pnpm dev

# Format code (Biome with Ultracite preset)
pnpm format

# Lint code
pnpm lint

# Type check
pnpm test-types

# Run tests
pnpm test
```

### Individual Package Development
Navigate to any package directory (e.g., `packages/core`) and run:
```bash
pnpm dev      # Development mode with watch
pnpm build    # Production build
pnpm types    # Generate declarations
```

### Publishing Workflow (via Changesets)
```bash
pnpm pub:changeset    # Create changeset for version bump
pnpm pub:version      # Apply changesets and bump versions
pnpm pub:release      # Publish to npm
```

## High-Level Architecture

### Monorepo Structure

This is a Turborepo monorepo with pnpm workspaces:
- `packages/*` - Core packages and plugins (11+ packages)
- `examples/*` - Example applications (dashboard, form)

Examples use `workspace:*` protocol to reference local packages during development.

### Core Package Architecture

The engine follows a **layered plugin architecture**:

1. **Core Layer** (`@easy-editor/core`)
   - Framework-agnostic engine foundation
   - Key modules: Designer, Document, Engine, Materials, Plugin system, Setters, Simulator
   - State management via MobX
   - Provides extension points for plugins

2. **Renderer Layer**
   - `@easy-editor/renderer-core` - Base renderer (framework-agnostic)
   - `@easy-editor/react-renderer` - React-specific implementation
   - Future: `vue-renderer`, `solid-renderer`, etc.

3. **Plugin Layer** (extends core functionality)
   - `@easy-editor/plugin-dashboard` - Dashboard building features
   - `@easy-editor/plugin-datasource` - Data source management
   - `@easy-editor/plugin-hotkey` - Keyboard shortcuts
   - `@easy-editor/plugin-form` - Form builder (WIP)

4. **Renderer Plugin Layer** (combines plugins + renderers)
   - `@easy-editor/react-renderer-dashboard` - Dashboard rendering in React
   - `@easy-editor/react-renderer-form` - Form rendering in React (WIP)

### Ecosystem Architecture

The EasyEditor ecosystem consists of 4 separate repositories:

```
EasyEditor (Core Engine - this repo)
    ├─> EasyMaterials (component library, independently packaged materials)
    ├─> EasySetters (property configuration UI, unified package)
    └─> EasyDashboard (reference application consuming published packages)
```

**Key Relationships**:
- **Materials** register with core's material system and can be loaded on-demand
- **Setters** register with core's setter system for property configuration
- **Applications** consume published `@easy-editor/*` packages from npm

### Plugin System Design

Plugins extend the engine through:
- **Lifecycle hooks** - Initialize, mount, unmount, etc.
- **Hotkey bindings** - Register keyboard shortcuts
- **Class extensions** - Extend core classes (Designer, Document, etc.)
- **Dependency injection** - Access shared services

### Material System

Materials are visual components that can be dragged into the canvas. Each material consists of:
- `component.tsx` - React component implementation
- `meta.ts` - Metadata (name, icon, category, etc.)
- `configure.ts` - Property configuration (uses setters)
- `snippets.ts` - Default instances/templates
- `constants.ts` - Component-specific constants

Materials can be:
- Bundled with the application
- Loaded dynamically from CDN
- Tree-shaken when unused

### Setter System

Setters are UI controls for configuring component properties in the designer:
- **Basic setters**: string, number, color, switch, upload, rect, node-id
- **Group setters**: collapse, tab (for organizing properties)
- Built with Tailwind CSS v4 + shadcn/ui for modern UX

### Remote Resource Loading Architecture

Remote materials and setters can be loaded dynamically from CDN. The architecture follows a strict separation:

**Core Layer** (`@easy-editor/core/remote/`):
- Pure TypeScript types and MobX state management
- NO browser-specific code (DOM APIs)
- Files: `types.ts`, `errors.ts`, `loading-state.ts`, `resource-registry.ts`

**Application Layer** (`examples/dashboard/src/editor/remote/`):
- Browser-specific implementations
- CDN loading with multi-provider fallback (unpkg, jsdelivr, fastly)
- Structure:
  ```
  remote/
  ├── loaders/           # Browser-specific loading
  │   ├── cdn-provider.ts
  │   ├── version-resolver.ts
  │   ├── script-loader.ts
  │   ├── material-loader.ts
  │   ├── setter-loader.ts
  │   └── local-loader.ts    # Local dev server HMR
  ├── managers/          # Business logic with MobX
  │   ├── material-manager.ts
  │   └── setter-manager.ts

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Easy-Editor/EasyEditor](https://github.com/Easy-Editor/EasyEditor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-31 -->
