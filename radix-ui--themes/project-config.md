---
trigger: always_on
description: > Context file for AI agents working in this repository.
---

# AGENTS.md — Radix Themes

> Context file for AI agents working in this repository.

## Project Overview

**Radix Themes** (`@radix-ui/themes`) is an open-source React component library optimized for fast development, easy maintenance, and accessibility. It provides a comprehensive set of pre-styled, themeable UI components built on top of [Radix Primitives](https://www.radix-ui.com/primitives).

- **Package name:** `@radix-ui/themes`
- **Current version:** 3.x (check `packages/radix-ui-themes/package.json` for exact version)
- **License:** MIT
- **Documentation:** https://www.radix-ui.com/themes/docs/theme

## Documentation

The full docs for this project are hosted at:

- **Theme overview:** https://www.radix-ui.com/themes/docs/theme/overview
- **Component docs:** https://www.radix-ui.com/themes/docs/components (e.g., `.../components/button`, `.../components/dialog`)
- **Utilities:** https://www.radix-ui.com/themes/docs/utilities/box
- **Releases:** https://www.radix-ui.com/themes/docs/overview/releases

When you need to understand a component's API, intended behavior, or usage patterns, **consult the docs at the URLs above** rather than guessing. The documentation site lives in a [separate repository](https://github.com/radix-ui/website).

## Repository Structure

This is a **pnpm monorepo** managed with **Turborepo**.

```
themes/
├── packages/
│   └── radix-ui-themes/          # The core library (published as @radix-ui/themes)
│       ├── src/
│       │   ├── components/       # All component source files
│       │   │   ├── _internal/    # Shared base components (not exported)
│       │   │   ├── *.tsx         # Component implementations
│       │   │   ├── *.props.tsx   # Component prop definitions
│       │   │   ├── *.css         # Component styles
│       │   │   └── index.tsx     # Public component exports
│       │   ├── helpers/          # Utility functions (prop extraction, responsive styles, etc.)
│       │   ├── props/            # Shared prop definitions (color, margin, radius, etc.)
│       │   ├── styles/
│       │   │   ├── tokens/       # Design tokens (colors, typography, radius, shadow, spacing)
│       │   │   └── utilities/    # CSS utility classes (layout, typography)
│       │   └── index.ts          # Package entry point
│       ├── scripts/              # Build scripts (esbuild CJS/ESM)
│       ├── postcss.config.js     # PostCSS config with custom plugins
│       ├── tsconfig.json
│       └── package.json
├── apps/
│   └── playground/               # Next.js App Router playground for development
│       ├── app/
│       │   ├── (themeable)/sink/ # Main component showcase (http://localhost:3000/sink)
│       │   ├── (themeable)/test/ # Test pages for specific scenarios
│       │   └── (themeable)/demo/ # Demo pages
│       ├── tests/                # Playwright visual regression tests
│       └── playwright.config.ts
├── .github/
│   ├── workflows/
│   │   ├── ci.yml                # Build, lint, format check, visual regression
│   │   └── publish.yml           # Publish to npm on GitHub release
│   └── CONTRIBUTING.md
├── package.json                  # Root monorepo config
├── pnpm-workspace.yaml
└── turbo.json
```

## Tech Stack

| Concern                | Tool                                                       |
| ---------------------- | ---------------------------------------------------------- |
| Package manager        | pnpm 10 (see `.npmrc`)                                     |
| Node version           | 24 (see `.nvmrc`)                                          |
| Monorepo orchestration | Turborepo                                                  |
| Language               | TypeScript (strict)                                        |
| Framework              | React (16.8+, 17, 18, 19 supported)                        |
| JS bundler             | esbuild (dual CJS + ESM output)                            |
| CSS processing         | PostCSS (nesting, custom-media, breakpoints, autoprefixer) |
| Linting                | ESLint (flat config), Stylelint (for CSS)                  |
| Formatting             | Prettier (single quotes, 100 print width, 120 for CSS)     |
| Playground             | Next.js (App Router)                                       |
| Testing                | Playwright (visual regression)                             |

## Key Commands

Run all commands from the **repository root** unless stated otherwise:

```bash
# Install dependencies
pnpm install

# Start development (playground + watch mode)
pnpm dev
# Then visit http://localhost:3000/sink

# Build the library
pnpm build

# Build only the themes package
pnpm build:pkg

# Lint everything
pnpm lint

# Format code
pnpm format

# Check formatting
pnpm format:check

# Run visual regression tests
pnpm test:vr

# Update visual regression snapshots
pnpm test:vr:update

# Clean all build artifacts
pnpm clean
```

## Component Architecture

### File Convention

Each component typically consists of three co-located files:

- **`component-name.tsx`** — React component implementation
- **`component-name.props.tsx`** — Prop type definitions and default values
- **`component-name.css`** — Component styles using CSS custom properties and data attributes


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [radix-ui/themes](https://github.com/radix-ui/themes) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
