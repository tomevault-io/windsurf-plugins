---
trigger: always_on
description: This document provides guidance for AI assistants working on the `@storybook-astro/framework` project. It covers architecture, conventions, and common development tasks.
---

# AGENTS.md - AI Development Guide

This document provides guidance for AI assistants working on the `@storybook-astro/framework` project. It covers architecture, conventions, and common development tasks.

## Project Overview

**Goal**: Enable Astro components to work in Storybook by implementing a custom Storybook framework integration.

**Status**: Experimental - not production-ready

**Key Technologies**:
- Astro 6+ (using Container API for SSR)
- Storybook 10+
- Vite 6+ (7.x supported)
- TypeScript/JavaScript (ES modules only)
- Multiple UI framework integrations (React, Vue, Svelte, Preact, Solid, Alpine.js)

## Architecture

### Two-Package System

#### 1. `packages/@storybook-astro/framework` (Server/Framework)
**Purpose**: Storybook framework definition and server-side rendering

**Key Responsibilities**:
- Configure Vite to handle Astro components
- Set up Astro Container for server-side rendering
- Manage framework integrations (React, Vue, etc.)
- Handle module resolution for Astro runtime

**Important Files**:
- `src/preset.ts` - Framework configuration, exports `viteFinal` and `core` config
- `src/middleware.ts` - Creates Astro Container, exports `handlerFactory`, includes `patchCreateAstroCompat` for Astro compiler v2/v3 bridging
- `src/viteStorybookAstroMiddlewarePlugin.ts` - Vite plugin that handles render requests via HMR
- `src/vitePluginAstroComponentMarker.ts` - Patches Astro 6's client-side `.astro` stubs to set `isAstroComponentFactory` and preserve scoped CSS imports
- `src/vitePluginAstroFontsFallback.ts` - Stubs Astro 6's font virtual modules (`virtual:astro:assets/fonts/*`)
- `src/portable-stories.ts` - `composeStories`/`composeStory` for testing outside Storybook
- `src/integrations/` - Integration adapters for each supported framework

#### 2. `packages/@storybook-astro/renderer` (Client)
**Purpose**: Client-side rendering logic in Storybook's preview iframe

**Key Responsibilities**:
- Render components in Storybook canvas
- Send render requests to server middleware
- Handle framework fallback rendering
- Manage styles and script hydration

**Important Files**:
- `src/render.tsx` - Exports `render()` and `renderToCanvas()` functions
- `src/preset.ts` - Defines preview annotations

### Data Flow

**Astro components** (server-side rendered):
```
Story File (.stories.jsx)
    ↓
@storybook-astro/renderer (render.tsx)
    ↓ [detects isAstroComponentFactory flag]
    ↓ [sends render request via Vite HMR]
@storybook-astro/framework (middleware.ts)
    ↓ [patchCreateAstroCompat wraps component]
    ↓ [Astro Container API renders to HTML]
@storybook-astro/renderer (render.tsx)
    ↓ [injects HTML into canvas]
    ↓ [applies scoped styles, executes client scripts]
Storybook Canvas (rendered component)
```

**Framework components** (React, Solid, Vue, etc. — delegated):
```
Story File (.stories.jsx)
    ↓
@storybook-astro/renderer (render.tsx)
    ↓ [checks parameters.renderer]
    ↓ [delegates to framework renderToCanvas BEFORE calling storyFn()]
Framework Renderer (e.g. @storybook/react-vite)
    ↓ [manages its own reactive root]
Storybook Canvas (rendered component)
```

## Code Conventions

### General
- **Module System**: ES modules only (`"type": "module"` in package.json)
- **File Extensions**: Use `.ts`, `.tsx`, `.js` explicitly in imports
- **Package Manager**: Yarn 4+ (Berry) with workspaces
- **Workspace Protocol**: Use `workspace:*` for internal package dependencies

### TypeScript
- TypeScript is used with proper types where possible
- `AstroRenderer` (extending `WebRenderer`) is the canonical renderer type used for Storybook generics
- Type definitions are in `types.ts` files in each package

### Naming
- Framework integration files: `packages/@storybook-astro/framework/src/integrations/[framework].ts`
- Vite plugins: Prefixed with `vite` or `vitePlugin`
- Virtual modules: Named like `virtual:astro-container-renderers`

### Imports
```typescript
// Good - explicit extension
import { handlerFactory } from './middleware.ts';

// Bad - no extension
import { handlerFactory } from './middleware';
```

## Common Development Tasks

### Adding a New Framework Integration

1. Create integration file: `packages/@storybook-astro/framework/src/integrations/[framework].ts`
2. Extend `BaseIntegration` class from `base.ts`
3. Implement required methods:
   - `getAstroRenderer()` - Returns Astro integration
   - `getVitePlugins()` - Returns Vite plugins for the framework
   - `getStorybookRenderer()` - Returns Storybook renderer name
   - `resolveClient()` - Handles client-side module resolution
4. Export factory function in `integrations/index.ts`
5. Add to `.storybook/main.js` configuration example

**Template**:
```typescript
import { BaseIntegration, type BaseOptions } from './base.ts';

export type Options = BaseOptions & {
  // Framework-specific options
};

export class FrameworkIntegration extends BaseIntegration {
  constructor(options?: Options) {
    super(options);
  }

  override getAstroRenderer() {
    // Return Astro framework integration
    return frameworkIntegration(/* config */);
  }

  override getVitePlugins() {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [storybook-astro/storybook-astro](https://github.com/storybook-astro/storybook-astro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
