---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Explorar.dev is a Next.js 16 application for exploring and learning from arbitrary software source code with an interactive VS Code-like interface. It supports both curated repositories (pre-downloaded at build time) and arbitrary GitHub repositories (downloaded on-demand).

**Key Architecture Decisions**:

- **Frontend**: Static Next.js export with no server-side rendering
- **No Backend**: Pure frontend application, no authentication or user data collection
- **Repository Modes**:
  - **Curated Mode**: Static repositories downloaded at build time, served from `/public/repos/`
  - **Arbitrary Mode**: User-selected GitHub repositories downloaded on-demand to IndexedDB

## Essential Commands

### Development

```bash
npm run dev              # Start dev server (port 3000)
```

**Environment Variable Configuration:**

- `.env.development` → Development defaults - loaded when `NODE_ENV=development`
- `.env.production` → Production defaults - loaded when `NODE_ENV=production`
- `.env.local` → Personal overrides (gitignored) - HIGHEST PRIORITY, overrides both above
- `npm run dev` automatically uses `.env.development` (Next.js sets `NODE_ENV=development`)
- `npm run build` automatically uses `.env.production` (Next.js sets `NODE_ENV=production`)

### Build & Test

```bash
# Development
npm run dev              # Start dev server with Turbopack (runs predev script first)
npm run predev           # Downloads Python/CPython repo for local testing

# Building & Deployment
npm run build            # Build static export (runs prebuild script)
npm run prebuild         # Downloads repositories based on --depth flag
npm run clean            # Remove all generated artifacts

# Code Quality
npm run lint   # Full gate: guide validation → tsc → eslint (cached) → prettier check → depcheck
npm run fix    # Auto-fix: eslint --fix + prettier write + npm audit fix + tsc verify
               # NOTE: fix is best-effort. Always run lint afterward to confirm clean state.
               # Pre-commit hook runs lint-staged (eslint + prettier) on changed files automatically.

# Testing
npm test                 # Run all Playwright tests
npm run test:sanity      # Basic functionality tests
npm run test:performance # Core Web Vitals tests
npm run test:seo         # SEO validation
npm run test:quality     # Accessibility checks (axe-core)
npm run test:score       # Quality score calculation
npm run test:ui          # Interactive test UI
npm run test:report      # View HTML test report

# Testing Notes
# - Tests require the static build: npm run build
# - Tests automatically start an http.server on port 8000
# - Configure test target with: BASE_URL=http://localhost:3000 npm test
```

## Architecture

### High-Level Data Flow

```
User → Route ([owner]/[repo]) → KernelExplorer (Main Container)
         ↓
      RepositoryContext (State Management)
         ↓
    ┌────┴────┬─────────────┬────────────┐
    ↓         ↓             ↓            ↓
 FileTree  CodeEditor  GuidePanel  DataStructuresView
    ↓         ↓             ↓            ↓
  API Layer (github-api.ts)
    ↓
  ┌─┴─────────────────────────────┐
  ↓                               ↓
Static (repo-static.ts)    Dynamic (github-archive.ts)
  ↓                               ↓
/public/repos/*            IndexedDB (repo-storage.ts)
(Curated)                  (Downloaded)
```

### Key Components

**UI Layer** (`src/components/`):

- `KernelExplorer.tsx`: Main container orchestrating all UI elements
- `FileTree.tsx`: Hierarchical file navigation
- `CodeEditorContainer.tsx`: Tab management for open files
- `MonacoCodeEditor.tsx`: Monaco editor wrapper with syntax highlighting
- `GuidePanel.tsx`: Learning guides with markdown parsing
- `DataStructuresView.tsx`: API/data structure browser
- `ActivityBar.tsx`: Navigation sidebar (files, guides, data structures)
- `TabBar.tsx`: Open file tabs
- `StatusBar.tsx`: Bottom status information
- `GitHubRateLimitWrapper.tsx`: Rate limit warnings

**State Management** (`src/contexts/`):

- `RepositoryContext.tsx`: Repository selection, branch management, download state
- `GitHubRateLimitContext.tsx`: Rate limit tracking

**Storage & Data Layer** (`src/lib/`):

- `repo-storage.ts`: IndexedDB wrapper - primary storage for downloaded repos
- `github-archive.ts`: GitHub API integration - downloads tree structure, lazy-loads files
- `repo-static.ts`: Static file reader - serves pre-built curated repos
- `github-api.ts`: File fetching, tree building, API dispatch (routes to static or dynamic)
- `github-cache.ts`: Caching layer with exponential backoff
- `github-retry.ts`: Retry logic and circuit breaker pattern

**Build & Platform** (`src/lib/`):

- `project-guides.tsx`: Configuration for curated repos (paths, guides)
- `monaco-workers.ts`: Monaco editor web worker setup
- `monaco-config.ts`: Editor theme and language configuration
- `platform/`: Platform abstraction (web vs. other environments)

### Repository Modes

**Curated (Static)**:

- Repositories pre-downloaded at build time via `scripts/download-repos.ts`
- Files live in `/public/repos/[owner]/[repo]/[branch]/`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pkill37/explorar.dev](https://github.com/pkill37/explorar.dev) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
