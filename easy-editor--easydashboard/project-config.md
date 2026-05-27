---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

EasyDashboard is a data visualization dashboard builder developed on top of the EasyEditor low-code engine. It serves as a reference application demonstrating how to build production applications using published `@easy-editor/*` packages.

## Build & Development Commands

```bash
# Install dependencies (pnpm 9.12.2+, node >= 18.0.0)
pnpm install

# Start development server (Vite)
pnpm dev

# Build for production
pnpm build

# Build for production (skip type checking)
pnpm build:prod

# Preview production build
pnpm preview

# Format code (Biome)
pnpm format

# Lint code (Biome)
pnpm lint

# Add shadcn/ui component
pnpm add:ui
```

## High-Level Architecture

### Application Type

This is a **single-page application (SPA)** built with:
- React 19 + Vite
- React Router for routing (`/` editor, `/preview` preview mode)
- Tailwind CSS v4 + shadcn/ui for UI components
- Monaco Editor for code editing

### Key Dependencies

**EasyEditor Core Packages** (consumed from npm):
- `@easy-editor/core` - Core engine
- `@easy-editor/plugin-dashboard` - Dashboard features
- `@easy-editor/plugin-datasource` - Data source management
- `@easy-editor/plugin-hotkey` - Keyboard shortcuts
- `@easy-editor/renderer-core` - Base renderer
- `@easy-editor/react-renderer` - React renderer
- `@easy-editor/react-renderer-dashboard` - Dashboard-specific React renderer

**UI Libraries**:
- `@radix-ui/*` - Accessible UI primitives
- `lucide-react` - Icons
- `recharts` - Chart library
- `@monaco-editor/react` - Code editor

**State Management**:
- `mobx-react` - For observing EasyEditor's MobX state

### Project Structure

```
src/
├── editor/                 # EasyEditor configuration
│   ├── index.ts           # Engine initialization
│   ├── const.ts           # Default project schema
│   ├── materials/         # Material registration
│   ├── plugins/           # Custom plugins
│   ├── setters/           # Setter registration
│   └── overrides.css      # Engine style overrides
│
├── pages/
│   ├── editor/            # Main editor page
│   └── preview/           # Preview mode page
│
├── components/            # Reusable UI components
│   ├── ui/               # shadcn/ui components
│   └── theme-provider.tsx # Dark mode support
│
├── lib/                   # Utilities
│   ├── schema.ts         # LocalStorage schema management
│   └── utils.ts          # Helper functions
│
├── hooks/                 # Custom React hooks
├── styles/                # Global styles
└── App.tsx               # App entry + routing
```

### Editor Initialization Flow

**Entry point**: `src/main.tsx` imports `src/editor/index.ts` before rendering React:

1. **Plugin Registration** (`src/editor/index.ts:13-27`)
   - Register `DashboardPlugin` with Group component configuration
   - Register `HotkeyPlugin`, `DataSourcePlugin`
   - Register custom plugins from `src/editor/plugins/`

2. **Material Registration** (`src/editor/index.ts:28`)
   - Build component metadata map from `src/editor/materials/componentMetaMap`
   - Materials are imported from the EasyEditor ecosystem

3. **Setter Registration** (`src/editor/index.ts:29`)
   - Register all setters from `src/editor/setters/setterMap`
   - Setters control property configuration UI

4. **Engine Initialization** (`src/editor/index.ts:31-38`)
   - Call `init()` with design mode and app helpers
   - Configure simulator viewport (1920x1080)

5. **Project Loading** (`src/editor/index.ts:44-54`)
   - Load project schema from localStorage if exists
   - Otherwise use `defaultProjectSchema` from `src/editor/const.ts`

### Material System Integration

Materials are registered in `src/editor/materials/`:
- Each material exports metadata conforming to EasyEditor's material spec
- Materials can be from `@easy-editor/materials-*` packages or custom implementations
- `componentMetaMap` aggregates all materials for registration

### Setter System Integration

Setters are registered in `src/editor/setters/`:
- Imports setters from `@easy-editor/setters` or custom implementations
- `setterMap` maps setter names to setter components
- Setters appear in the property configuration panel

### Custom Plugins

Custom plugins in `src/editor/plugins/` extend editor functionality:
- Follow EasyEditor's plugin API
- Register via `plugins.registerPlugins()`
- Can access core services via dependency injection

## Vite Configuration

Key settings in `vite.config.mts`:

- **React Plugin**: Babel configured with decorators support (for MobX)
- **Tailwind Plugin**: `@tailwindcss/vite` for v4 support
- **Path Alias**: `@/` maps to `./src`
- **Build Target**: `esnext` for modern browsers

## TypeScript Configuration

Uses TypeScript project references:
- `tsconfig.json` - Root configuration with path aliases (`@/*`)
- `tsconfig.app.json` - App source code settings
- `tsconfig.node.json` - Vite config and Node scripts

Path alias `@/*` maps to `./src/*` for cleaner imports.

## Code Standards

Uses **Biome** for formatting and linting:

### Key Rules
- **Type Safety**: Explicit types, avoid `any`
- **Modern React**: Function components, hooks, proper dependencies

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Easy-Editor/EasyDashboard](https://github.com/Easy-Editor/EasyDashboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-27 -->
