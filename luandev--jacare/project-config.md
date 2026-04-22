---
trigger: always_on
description: Jacare (Portuguese for "caiman") is an open-source, web-based desktop ROM library manager built as an Electron shell over an Express API and a React UI. The application helps users browse, search, and manage their retro game collection with persistent download management, customizable themes, and intelligent metadata enrichment.
---

# Jacare (CrocDesk) - Copilot Instructions

## Project Overview

Jacare (Portuguese for "caiman") is an open-source, web-based desktop ROM library manager built as an Electron shell over an Express API and a React UI. The application helps users browse, search, and manage their retro game collection with persistent download management, customizable themes, and intelligent metadata enrichment.

**Key Features:**
- All-in-one solution for browsing and managing ROM collections
- Persistent download management with pause/resume capabilities
- Customizable themes and ultra-responsive web-based UI
- Local-first design with cloud metadata from Crocdb API
- Smart synchronization and automatic library management

## Architecture

The project follows a monorepo structure with three main applications and a shared package:

```
┌─────────────────────────────────────────────────┐
│           Electron Desktop App                  │
│         (apps/desktop)                          │
│  ┌───────────────────────────────────────────┐  │
│  │  Express API Server (apps/server)         │  │
│  │  - REST endpoints                         │  │
│  │  - SSE for real-time job updates          │  │
│  │  - Crocdb client & caching                │  │
│  │  - Job orchestration                      │  │
│  │  - SQLite data storage                    │  │
│  └───────────────────────────────────────────┘  │
│  ┌───────────────────────────────────────────┐  │
│  │  React UI (apps/web)                      │  │
│  │  - Vite + React                           │  │
│  │  - TanStack Query for API state           │  │
│  │  - SSE client for job progress            │  │
│  └───────────────────────────────────────────┘  │
└─────────────────────────────────────────────────┘
         ┌────────────────────┐
         │  Shared Types      │
         │  (packages/shared) │
         └────────────────────┘
```

**Design Principles:**
- Server can run standalone (headless mode) or embedded in Electron
- All long-running operations execute as background jobs
- Real-time updates via Server-Sent Events (SSE)
- Local-first with external metadata fetching and caching

## Repository Layout

- **`apps/server/`** - Express API, job orchestration, local scanning, Crocdb client
  - `src/` - TypeScript source code
  - `scripts/` - Utility scripts
  - Dockerfile for containerized deployment
- **`apps/web/`** - React UI built with Vite
  - `src/` - React components and application logic
  - Uses TanStack Query for API state management
- **`apps/desktop/`** - Electron main process
  - `src/` - Electron entry point and window management
  - `build/` - Build resources for packaging
- **`packages/shared/`** - Shared TypeScript types, defaults, and manifest schema
  - `src/` - Shared type definitions and utilities
- **`tests/`** - Root-level test setup and fixtures
- **`docs/`** - Developer and user documentation
- **`docker/`** - Docker Compose templates and configuration

## Development Workflow

### Prerequisites
- Node.js 20+ (includes npm)
- Git

### Installation
```bash
npm ci
```

### Development Commands

**Start everything in development mode:**
```bash
npm run dev  # Runs shared build watch + server + web + desktop concurrently
```

**Start individual workspaces:**
```bash
npm run dev:shared   # Shared package watch mode
npm run dev:server   # Express API + jobs
npm run dev:web      # React UI with Vite (default port: 5173)
npm run dev:desktop  # Electron shell
```

**Build, type-check, lint, and test:**
```bash
npm run build       # Build all workspaces
npm run typecheck   # Type-check the monorepo
npm run lint        # Lint all code with ESLint
npm run test:unit   # Run unit tests with Vitest
npm run test        # Alias for test:unit
```

**Packaging commands:**
```bash
npm run package:bundle   # Create standalone binaries (Windows, macOS, Linux)
npm run package:desktop  # Package Electron desktop app
npm run package:server   # Package server-only binary
```

### Testing Strategy

- **Unit tests:** Vitest with JSDOM for React components
- **Test location:** `apps/**/src/**/__tests__/**/*.{test,spec}.{ts,tsx}` or `apps/**/src/**/*.{test,spec}.{ts,tsx}`
- **Test environment:** Node.js by default, JSDOM for `apps/web/**`
- **Test setup:** `tests/vitest.setup.ts`
- **Coverage:** v8 provider with text and HTML reports

**When adding tests:**
- Place tests near the code they test (prefer `__tests__` directory or `.test.ts` suffix)
- Use Vitest's `describe`, `it`, `expect` API
- Mock external dependencies (Crocdb API, file system operations)
- Test both success and error cases

### Linting Rules

- ESLint with TypeScript parser
- React-specific rules for `apps/web/**`
- Unused variables allowed if prefixed with `_`
- `@typescript-eslint/no-explicit-any` disabled (use sparingly)
- React components don't require JSX scope import (React 17+ transform)
- Accessibility rules from `jsx-a11y` (some relaxed for UI convenience)

**When writing code:**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [luandev/jacare](https://github.com/luandev/jacare) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
