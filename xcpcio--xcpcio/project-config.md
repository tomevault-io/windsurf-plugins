---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**XCPCIO** is "The ICPC Series Competition Leaderboard Visualization Engine" - a comprehensive platform for ICPC competitive programming contests. It provides professional ranking systems, real-time leaderboard visualization, and hosting services for programming competitions.

## Architecture

This is a pnpm monorepo with the following structure:

- **packages/libs/types**: TypeScript type definitions for the entire project
- **packages/libs/core**: Core business logic, utilities, and algorithms for contest processing
- **packages/apps/board**: Vue 3 frontend application for displaying contest leaderboards
- **python/**: Python library providing the same type definitions and data models as the TypeScript packages

The project follows a layered architecture where:

### npm

- `@xcpcio/types` provides shared type definitions
- `@xcpcio/core` contains contest logic, ranking algorithms, and data processing
- `@xcpcio/board-app` is the main Vue application consuming the core libraries

### python

- `xcpcio` (Python) provides equivalent type definitions and data models for Python environments

## Development Commands

### Root Level Commands

```bash
# Install dependencies
pnpm install

# Build all packages
pnpm build

# Build only libraries (types and core)
pnpm build:libs

# Build docs app
pnpm build:docs

# Build board app
pnpm build:board

# Run tests
pnpm test

# Update test snapshots
pnpm test:update

# Run full CI pipeline (build, lint, test)
pnpm test:ci

# Lint all code
pnpm lint

# Fix lint issues automatically
pnpm lint:fix

# Start docs development server
pnpm docs

# Start board development server
pnpm board

# Release new version (bumps version, commits, tags, pushes)
pnpm release

# Publish to npm (requires build first)
pnpm publish:npm

# Build and publish to npm
pnpm publish:npm_with_build
```

### Package-Specific Commands

#### Board App (packages/apps/board)

```bash
# Development server on port 3333
pnpm dev

# Build for production
pnpm build

# Preview production build
pnpm preview

# Preview with HTTPS
pnpm preview-https

# Start production server
pnpm start

# Run E2E tests with Cypress
pnpm test:e2e

# Bundle size analysis
pnpm sizecheck
```

#### Core Library (packages/libs/core)

```bash
# Build library
pnpm build

# Development mode with stub
pnpm dev

# Run directly with esmo
pnpm start
```

#### Types Library (packages/libs/types)

```bash
# Build library
pnpm build

# Development mode with stub
pnpm dev
```

#### Python Library (python/)

```bash
# Install dependencies with uv
uv install

# Run tests
uv run pytest

# Run specific test file
uv run pytest tests/test_types.py

# Run with coverage
uv run pytest --cov=xcpcio

# Format code with ruff
uv run ruff format

# Lint code with ruff
uv run ruff check
```

## Technology Stack

- **Frontend**: Vue 3, TypeScript, Vite, UnoCSS, Pinia for state management
- **UI Components**: Floating Vue for tooltips, Flowbite for components, Vue Search Select
- **Data Visualization**: Highcharts with Vue integration, GSAP for animations
- **State Management**: Pinia, Vue Query (@tanstack/vue-query) for server state
- **Utilities**: VueUse composables, Day.js for dates, Lodash for utilities
- **Build System**: Vite for apps, unbuild for libraries, Vite SSG for static generation
- **Testing**: Vitest for unit tests, Cypress for E2E, pytest for Python
- **Linting**: ESLint with @antfu/eslint-config, Ruff for Python
- **Package Management**: pnpm with workspace configuration and catalog, uv for Python
- **Python**: Pydantic for data validation and serialization
- **Documentation**: VitePress for documentation site

## Key Concepts

### Contest Data Structure

The core revolves around contest data represented by the `Contest` interface in `packages/libs/types/src/contest.ts` and the equivalent Pydantic model in `python/xcpcio/types.py`. This includes:

- Contest metadata (name, times, organization)
- Problems and balloon colors
- Team rankings and submissions
- Medal configurations and groupings

### Python Library

The Python library (`python/xcpcio/`) provides:

- **Type Definitions**: Pydantic models mirroring the TypeScript types for contests, teams, submissions
- **Constants**: Shared constants for submission statuses, time units, penalty calculations
- **Data Validation**: Built-in validation and serialization using Pydantic
- **Cross-Language Compatibility**: Ensures data consistency between Python and TypeScript environments

Key models include:

- `Contest`: Contest configuration and metadata
- `Team`: Team information with groups, tags, and extra fields
- `Submission`: Individual submission data with status and timing

### Ranking System

The ranking logic in `packages/libs/core/src/rank.ts` handles:

- ICPC-style scoring with penalties
- Real-time leaderboard updates
- Frozen standings during contest end
- Medal allocation based on presets or custom rules

### Vue Application

The board app uses:

- Vue Router with auto-generated routes
- Vite SSG for static site generation
- Vue Query for data fetching
- Floating Vue for tooltips and popovers

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xcpcio/xcpcio](https://github.com/xcpcio/xcpcio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
