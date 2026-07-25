---
trigger: always_on
description: This file provides guidance to AI agents when working with code in this repository.
---

# AGENTS.md

This file provides guidance to AI agents when working with code in this repository.

## Overview

Vocs is a **portable documentation framework powered by Vite**. This is Vocs v2, a ground-up rewrite and successor to [Vocs v1](https://vocs.dev), designed to be the best-in-class documentation framework for technical projects.

### Vision

Vocs v2 aims to be:

- **Portable** – Built as a Vite plugin, Vocs can be plugged into any Vite-based application. No framework lock-in.
- **Modern** – First-class React Server Components support via Waku, MDX with advanced code highlighting (Shiki + Twoslash), and Tailwind CSS.
- **Developer-first** – Exceptional DX with hot reloading, TypeScript-first APIs, and minimal configuration.
- **Performant** – Static site generation, optimized bundles, and smart caching for Twoslash.
- **Extensible** – Modular architecture allows customization at every layer.

When contributing, reflect these principles: prefer Vite plugins over custom CLIs; prefer RSC-compatible patterns when adding React features.

### Competitive Landscape

Vocs is an alternative to documentation frameworks like Docusaurus, Nextra, VitePress, Starlight, and Fumadocs. It differentiates by being Vite-native and framework-agnostic while providing first-class Waku integration.

When in doubt, reference `_/` (reference implementations of competing frameworks and Vocs v1) for patterns rather than reinventing APIs.

### Project Status

This project is **heavily in progress**. See `ROADMAP.md` for the full feature checklist.

- Prefer work aligned with `ROADMAP.md` over ad hoc features
- Avoid breaking v1 parity unless the roadmap indicates a deliberate change

## Commands

```sh
# Development
pnpm dev:playground    # Start playground dev server for testing

# Building & Type Checking
pnpm build             # Build the library using zile
pnpm check:types       # Typecheck with tsc

# Code Quality
pnpm check             # Lint + format with Biome

# Testing
pnpm test              # Run all tests (vitest ./src)
pnpm vitest src/internal/sidebar.test.ts  # Run a single test file
```

### When to Run

- Before any PR or large change: `pnpm check && pnpm test`
- For type-driven refactors: run `pnpm check:types` frequently
- For changes affecting the docs site: run `pnpm dev:playground` and verify manually

## Architecture

Vocs is built as a Vite plugin with optional Waku (React Server Components) integration. This separation keeps the core framework-agnostic while enabling RSC features when desired.

### Directory Structure

| Directory | Purpose | When to Add Here |
|-----------|---------|------------------|
| `src/internal/` | Core logic: config, MDX, Shiki, Vite plugins | Framework-agnostic features, config changes, MDX transforms |
| `src/react/` | React components (Layout, Callout, Link, etc) | User-facing components |
| `src/react/internal/` | MDX-specific components (CodeBlock, Steps, etc) | Components only used within MDX |
| `src/server/` | Server utilities | Server-side logic |
| `src/waku/` | Waku framework integration: router, middleware, plugins | RSC-specific behavior |
| `src/styles/` | Tailwind CSS styles | Design system changes |
| `playground/` | Development playground (real Vocs site) | Testing features end-to-end |
| `_/` | Reference implementations (read-only) | Do not modify; use for reference only |

### Entry Points

| File | Purpose | Extension Pattern |
|------|---------|-------------------|
| `src/vite.ts` | Core Vite plugin (MDX, Tailwind, icons, config) | New core plugins compose into the array returned here |
| `src/waku/vite.ts` | Waku-specific Vite plugin, wraps core + adds RSC | Waku-only plugins go here |
| `src/index.ts` | Public API exports | Only stable, public APIs; keep experimental APIs internal |
| `src/config.ts` | Public config export (`defineConfig`) | Config schema changes |

### Key Internal Modules

| Module | Responsibility | Invariants |
|--------|----------------|------------|
| `internal/config.ts` | Config resolution, serialization, global state | Always go through `Config`/`defineConfig`; never read raw user config directly |
| `internal/mdx.ts` | MDX compile options, remark/rehype plugins | Add new MDX plugins here, not scattered in Vite plugins |
| `internal/vite-plugins.ts` | Vite plugins for MDX, virtual config, route watching | All Vocs-specific Vite behavior encapsulated here |
| `internal/sidebar.ts`, `internal/topNav.ts` | Navigation parsing from config | Handle nested items, collapsing, active state |
| `internal/shiki-transformers.ts` | Shiki code highlighting transformers | Code block styling, line highlighting, diffs |
| `internal/twoslash/` | Twoslash integration with caching | TypeScript code examples with type hints |

### Virtual Modules

| Module | Purpose | Usage |
|--------|---------|-------|
| `virtual:vocs/config` | Serialized config, hot-reloads in dev | Import from Vite app side / RSC environment only |
| `virtual:vocs/user-styles` | User's `_root.css` from pages directory | Injected automatically |

When adding new user-extensible entrypoints, create a `virtual:vocs/*` module and wire it via `internal/vite-plugins.ts`.

## Code Style

### Module Pattern


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wevm/vocs](https://github.com/wevm/vocs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
