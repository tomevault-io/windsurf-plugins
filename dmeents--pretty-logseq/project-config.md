---
trigger: always_on
description: This file provides guidance to Claude Code when working with this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code when working with this repository.

## Project Overview

**Pretty Logseq** is a Logseq plugin for frontend customizations. It provides a
modular, settings-driven system of self-contained features: custom hover
popovers, enhanced external links, styled properties/tables/tags/templates/todos,
typography, and top bar / sidebar tweaks. Each feature can be toggled
independently from the plugin settings UI.

The plugin supports **both Logseq editions**: v1 (the file/markdown app) and v2
(the newer database-backed "DB" app). The two differ in DOM structure, CSS
variables, and the shape of data returned by `Editor.getPage`, so a cross-version
**platform adapter** (`src/core/platform/`) abstracts those differences behind a
single `Platform` interface. All v2 work is strictly **additive** — v1 behavior
must never regress (see the platform adapter section below).

## Tech Stack

- **Language:** TypeScript
- **Build:** Vite with vite-plugin-logseq
- **Styling:** SCSS (compiled via Vite, imported with `?inline`)
- **Testing:** Vitest (jsdom environment)
- **Linting/Formatting:** Biome
- **Package Manager:** pnpm 10.x
- **Runtime:** Logseq Plugin API (@logseq/libs)

## Commands

```bash
# Install dependencies
pnpm install

# Development (watch mode — rebuilds dist/ on change)
pnpm dev

# Production build
pnpm build

# Type check (no emit)
pnpm typecheck

# Tests
pnpm test              # Run tests once
pnpm test:coverage     # Run tests with coverage report

# Lint and format (Biome)
pnpm check             # Check + auto-fix
pnpm check:ci          # Check only (no writes) — used in CI
```

Note: there are no separate `lint`/`format` scripts — `check` covers both.

## Development Workflow

1. Run `pnpm dev` to start watching for changes
2. In Logseq: Settings → Advanced → Developer mode (enable)
3. In Logseq: Plugins → Load unpacked plugin → Select this project folder (not dist/)
4. Changes auto-rebuild; reload plugin in Logseq to see updates

## Build Configuration

The plugin uses Vite with vite-plugin-logseq. Key points:

- **Entry point:** `index.html` in project root loads `src/index.ts`
- **Output:** `dist/index.html` + `dist/assets/index-*.js`
- **package.json main:** Must be `"dist/index.html"` (not .js)
- **@logseq/libs:** Must be bundled (NOT external) - the browser needs the library code to set up the global `logseq` object
- **SCSS:** Uses the modern-compiler API (see `vite.config.ts`)

```
Build output:
dist/
├── index.html           # Entry point Logseq loads
└── assets/
    └── index-*.js       # ESM bundle (includes @logseq/libs + compiled CSS)
```

## Project Structure

```
pretty-logseq/
├── index.html            # Vite entry point (loads src/index.ts)
├── manifest.json         # Logseq plugin manifest
├── package.json          # Plugin manifest and dependencies
├── vite.config.ts        # Vite build configuration
├── vitest.config.ts      # Vitest configuration (jsdom, coverage)
├── biome.json            # Linting and formatting config
├── tsconfig.json         # TypeScript configuration
├── pnpm-workspace.yaml   # pnpm configuration
├── src/
│   ├── index.ts          # Bootstrap, feature registration, settings-change wiring
│   ├── types/            # TypeScript interfaces
│   │   ├── index.ts      # Re-exports
│   │   ├── feature.ts    # Feature / ConfigurableFeature interfaces
│   │   ├── logseq.ts     # PageProperties, etc.
│   │   └── scss.d.ts     # SCSS import declarations
│   ├── core/             # Core infrastructure
│   │   ├── registry.ts   # Feature lifecycle + style aggregation
│   │   ├── styles.ts     # Style injection (theme + features → provideStyle)
│   │   ├── theme.ts      # Accent-color auto-detection + theme observer
│   │   ├── version.ts    # Logseq edition detection (v1 vs v2), getVersion()
│   │   └── platform/     # Cross-version adapter (selectors, api, theme per edition)
│   │       ├── index.ts  # getPlatform() — resolves active Platform, pickStyles()
│   │       ├── types.ts  # Platform interface
│   │       ├── v1.ts     # v1 (file/markdown) platform — source of truth
│   │       ├── v2.ts     # v2 (DB) platform — spreads v1, overrides confirmed diffs
│   │       └── styles.ts # Version-aware style selection helpers
│   ├── lib/              # Shared utilities
│   │   ├── dom.ts        # Positioning, element creation, getParentDoc
│   │   ├── api.ts        # v1 Logseq API helpers with caching
│   │   └── api.v2.ts     # v2 (DB) data adapter — normalizes DB property shape
│   ├── settings/         # Plugin settings
│   │   ├── index.ts      # getSettings / initSettings / onSettingsChanged
│   │   └── schema.ts     # Settings UI schema + PluginSettings interface
│   └── features/         # Feature modules (each implements Feature)
│       ├── popovers/     # Custom hover previews for page references
│       │   ├── index.ts        # Feature entry point
│       │   ├── manager.ts      # Hover lifecycle, show/hide, positioning
│       │   ├── styles.scss     # Popover + native-suppression styles
│       │   └── renderers/
│       │       ├── index.ts        # Re-exports renderPopover
│       │       ├── unified.ts      # Config-driven renderer for all page types

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dmeents/pretty-logseq](https://github.com/dmeents/pretty-logseq) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
