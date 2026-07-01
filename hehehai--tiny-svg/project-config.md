---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Tiny SVG is a monorepo containing:
1. **Web app** - SVG optimizer and code generator (TanStack Start + React 19)
2. **Figma plugin** - SVG optimization directly in Figma
3. **Shared packages** - Core SVG logic, code generators, UI components

**Tech Stack:**
- Framework: TanStack Start (SSR with file-based routing), React 19
- UI: Tailwind CSS 4, Radix UI, shadcn/ui
- State: Zustand
- i18n: Intlayer (EN, ZH, KO, DE, FR) - web app only
- Build: Vite 7, pnpm workspaces (pnpm@10.14.0)
- Linting: Biome + Ultracite
- Deployment: Vercel (Cloudflare Workers not supported due to MDX `eval()` restrictions)

## Common Commands

```bash
# Development
pnpm dev              # Start all workspace apps
pnpm dev:web          # Start only web app (port 3001)
pnpm dev:figma        # Start Figma plugin in watch mode

# Build
pnpm build            # Build all packages (runs workspace builds in dependency order)
pnpm build:packages   # Build only shared packages (svg, code-generators, utils, ui)
pnpm build:figma      # Build Figma plugin for production
pnpm --filter @tiny-svg/web build   # Build web app only
pnpm --filter @tiny-svg/web serve   # Preview web app production build

# Code Quality
pnpm check            # Run Biome linter/formatter (auto-fix)
pnpm check-types      # TypeScript type checking across all workspaces

# Internationalization (web app only)
pnpm exec intlayer build  # Build i18n dictionaries (run from apps/web)
```

## Workspace Architecture

```
tiny-svg/
├── apps/
│   ├── web/                     # Main web application (TanStack Start)
│   └── figma-plugin/            # Figma plugin (React + Figma Plugin API)
└── packages/
    ├── svg/                     # @tiny-svg/svg - SVGO config, presets, utilities
    ├── code-generators/         # @tiny-svg/code-generators - Framework code generators
    ├── ui/                      # @tiny-svg/ui - Shared React components (diff viewer, etc.)
    └── utils/                   # @tiny-svg/utils - General utilities
```

### Package Dependencies

**Apps depend on packages:**
- `apps/web` → `@tiny-svg/svg`, `@tiny-svg/code-generators`, `@tiny-svg/ui`
- `apps/figma-plugin` → `@tiny-svg/svg`, `@tiny-svg/code-generators`, `@tiny-svg/ui`, `@tiny-svg/utils`

**Important:** After modifying any `packages/*`, run `pnpm build:packages` before building apps.

### Shared Packages

**`@tiny-svg/svg`** (`packages/svg/src/`)
- SVGO default configuration and presets (default-config.ts, presets.ts)
- Compression presets: Default, Aggressive, Minimal, Custom
- SVGO utilities and types
- Built with `tsdown` to CommonJS + ESM

**`@tiny-svg/code-generators`** (`packages/code-generators/src/`)
- Framework code generators: React (JSX/TSX), Vue, Svelte, React Native, Flutter
- Single file: `generators.ts` with all generator functions
- Built with `tsdown` to CommonJS + ESM

**`@tiny-svg/ui`** (`packages/ui/src/`)
- Shared React components (not built, consumed as source via workspace protocol)
- Diff viewer component with syntax highlighting
- Radix UI-based components shared between web and plugin
- Exports paths: `@tiny-svg/ui/components/*`, `@tiny-svg/ui/lib/*`

**`@tiny-svg/utils`** (`packages/utils/src/`)
- General utilities used across projects
- Built with `tsdown` to CommonJS + ESM

## Web App Architecture (`apps/web/`)

```
apps/web/src/
├── components/         # React components (UI, optimize, lazy-loaded)
├── contents/           # i18n definitions (*.content.ts)
├── hooks/              # Custom React hooks
├── lib/                # Utilities and helpers
│   ├── worker-utils/       # Worker client interfaces
│   ├── svg-to-code.ts      # Framework code generators (wraps @tiny-svg/code-generators)
│   ├── svg-transform.ts    # SVG transformations (rotate, flip, resize)
│   ├── svgo-plugins.ts     # SVGO plugin configurations (wraps @tiny-svg/svg)
│   └── file-utils.ts       # File handling utilities
├── routes/             # TanStack Start file-based routing
│   ├── __root.tsx          # Root layout
│   ├── {-$locale}/         # Locale-prefixed routes (e.g., /en, /zh)
│   └── og.tsx              # Open Graph image generation
├── store/              # Zustand stores
│   ├── svg-store.ts        # SVG content, optimization settings, transformations
│   └── ui-store.ts         # UI state (theme, panels, preferences)
└── workers/            # Web Workers for heavy tasks
    ├── svgo.worker.ts      # SVG optimization (SVGO runs here)
    ├── prettier.worker.ts  # Code formatting (Prettier runs here)
    └── code-generator.worker.ts  # Framework code generation
```

## Figma Plugin Architecture (`apps/figma-plugin/`)

```
apps/figma-plugin/
├── assets/             # Plugin assets
│   ├── icon.svg            # Vector icon (40x40)
│   └── icon.png            # Raster icon (128x128)
├── src/
│   ├── plugin.ts       # Plugin sandbox code (Figma API, runs in restricted context)
│   └── ui/             # React UI (runs in browser context)
│       ├── components/     # React components (svg-list, svg-item, preview-drawer, etc.)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hehehai/tiny-svg](https://github.com/hehehai/tiny-svg) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
