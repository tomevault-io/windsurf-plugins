---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

React Creme is a modern UI toolkit for React featuring 55+ high-quality, accessible, and themeable components. The project is built as a Turborepo monorepo with pnpm workspaces, containing a component library, documentation site, and Storybook for component development.

### Modern Tooling (2025)

This project uses cutting-edge tooling and configurations:

- **Build**: Vite 6.3.5 with LightningCSS for CSS optimization
- **Linting**: ESLint 9+ flat config + Oxlint (50-100x faster)
- **Testing**: Vitest 3.1.4 with jsdom
- **TypeScript**: 5.9.3 with strict mode and modern compiler options
- **Package Manager**: pnpm 10.14.0
- **Monorepo**: Turborepo 2.5.8
- **Code Quality**: Knip for unused code detection
- **CI/CD**: GitHub Actions with pnpm caching
- **Node**: v20.18.1 LTS (see `.nvmrc`)

## Repository Structure

```
react-creme/
├── packages/
│   ├── lib/              # Main component library (react-creme npm package)
│   ├── documentation/    # Documentation website (Vite-based)
│   └── storybook/        # Storybook component development environment
└── .storybook/           # Shared Storybook configuration
```

### Component Library (`packages/lib/`)

- **Components**: Each component lives in `components/<component-name>/` with its TypeScript implementation, SCSS module, and tests
- **Design System**: Global styles, tokens, and mixins are in `design/` (theme.scss, tokens.scss, effects.scss, etc.)
- **Entry Point**: `react-creme.ts` exports all public components and types
- **Build System**: Uses Vite (migrated from Webpack) with CSS Modules support

## ⚠️ Important: Monorepo Tooling

This is a **Turborepo monorepo**. All modern tooling (oxlint, knip, ESLint) is **monorepo-aware** and works across all packages.

**See `MONOREPO_TOOLING.md` for detailed architecture and workflow.**

### First Time Setup

```bash
# Install all dependencies (including oxlint, knip, lightningcss)
pnpm install

# Verify installation
pnpm oxlint --version
pnpm knip --version
```

## Common Development Commands

### Building

```bash
# Build everything (library, docs, storybook)
pnpm build

# Build only the library
pnpm build:lib
# OR from packages/lib:
pnpm build

# Build documentation
pnpm build:doc

# Build Storybook
pnpm build-story
```

### Development

```bash
# Run documentation dev server
pnpm dev

# Run Storybook dev server
pnpm story

# Run library in dev mode (from packages/lib)
cd packages/lib && pnpm dev
```

### Testing

```bash
# Run all tests
pnpm test

# Run tests in CI mode (from packages/lib)
cd packages/lib && pnpm test:ci

# Run tests with UI (from packages/lib)
cd packages/lib && pnpm test-ui

# Run a single component test (from packages/lib)
cd packages/lib && vitest --run components/button
```

### Linting & Formatting (Monorepo-Wide)

```bash
# Run all linters across all packages (oxlint → ESLint)
pnpm lint

# Run fast linting only (oxlint) - <1 second!
pnpm oxlint

# Format all packages
pnpm format

# Fix JavaScript/TypeScript across all packages
pnpm fix-js

# Fix CSS/SCSS across all packages
pnpm fix-css

# Fix everything
pnpm fix-all

# Run full quality check
pnpm clean

# Check for unused code/dependencies (monorepo-aware)
pnpm knip
pnpm knip:production  # Production dependencies only

# Run with Turbo filters (specific packages)
pnpm lint --filter=react-creme        # Lib only
pnpm lint --filter=*docu*             # Documentation only
pnpm oxlint --filter=react-creme      # Fast lint lib only
```

**How it works:**

- Turbo orchestrates tasks across packages in parallel
- Oxlint runs first (50-100x faster) as a pre-check
- ESLint runs after for comprehensive checking
- Results are cached for faster subsequent runs
- See `MONOREPO_TOOLING.md` for architecture details

### Running Single Package Commands

Since this is a Turborepo monorepo, you can run commands in specific packages:

```bash
# From root, using turbo filters
turbo run build --filter=react-creme        # library only
turbo run dev --filter=*docu*               # documentation only
turbo run dev --filter=*story*              # storybook only

# Or navigate to the package
cd packages/lib && pnpm build
cd packages/documentation && pnpm dev
cd packages/storybook && pnpm dev
```

## Architecture & Key Patterns

### Component Structure

Each component follows a consistent pattern:

```
components/<component-name>/
├── <component-name>.tsx           # Main component implementation
├── <component-name>-model.ts      # TypeScript types/interfaces
├── <component-name>.module.scss   # Component styles (CSS Modules)
└── __tests__/
    └── <component-name>.test.tsx  # Vitest tests
```

### CSS Modules

- **Naming Convention**: `[name]__[local]___[hash:base64:5]`
- **Import Pattern**: `import styles from './component.module.scss'`
- **Usage**: Classes are camelCased in TypeScript (e.g., `styles.buttonWrapper`)
- **Global Styles**: Design tokens and utilities are in `packages/lib/design/`

### Theme System

Components support theming through the `ThemeProvider`:

- Theme colors, sizes, and design tokens are defined in `design/theme.scss` and `design/tokens.scss`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [prabhuignoto/react-creme](https://github.com/prabhuignoto/react-creme) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
