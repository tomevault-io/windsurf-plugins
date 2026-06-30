---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

React Chrono is a modern, flexible timeline component library for React that supports multiple layout modes (horizontal, vertical, vertical-alternating, horizontal-all) with rich media support, theming, and accessibility features. Built with TypeScript, Vanilla Extract CSS, and tested with Vitest + Playwright.

**Compatibility:** React 18.2+ and React 19+ | Node.js 22+ | Modern browsers (Chrome, Firefox, Safari, Edge)

## Essential Commands

### Development
```bash
pnpm run dev              # Start dev server with hot reload (site preview)
pnpm run build            # Production build (generates dist/)
pnpm run build:analyze    # Build with bundle analysis
```

**Demo & Examples:**
- Demo site code: `src/demo/` (runs with `pnpm run dev`)
- Example components: `src/examples/`
- CodesandBox example: `codesandbox-example/`

### Testing
```bash
# Unit Tests (Vitest)
pnpm test                 # Run unit tests in watch mode
pnpm test -- --run        # Run tests once without watch mode
pnpm test timeline        # Run tests matching "timeline" in filename
pnpm run test-ui          # Run tests with Vitest UI

# E2E Testing (Playwright) - Cross-browser (Chromium, Firefox, WebKit)
pnpm run test:e2e         # Run all E2E tests (all browsers)
pnpm run test:e2e:ui      # Run E2E with Playwright UI (interactive debugging)
pnpm run test:e2e:headed  # Run E2E in headed mode (visible browser)
pnpm run test:e2e:debug   # Run E2E in debug mode (step-through)
pnpm run test:e2e:chrome  # Run E2E in Chromium only
pnpm run test:e2e:firefox # Run E2E in Firefox only
pnpm run test:e2e:webkit  # Run E2E in WebKit (Safari) only

# Component Tests (Playwright) - Cross-browser isolation testing
pnpm run test:ct          # Component tests with Playwright
pnpm run test:ct:ui       # Component tests with Playwright UI

# Integration Tests - Tests built package in real browser
pnpm run test:integration        # Run integration tests (validates dist/ works)
pnpm run test:integration:ui     # Run integration tests with UI
pnpm run test:integration:headed # Run integration tests in headed mode

# Test Utilities
pnpm run test:install     # Install Playwright browsers (required once)
pnpm run test:report      # View Playwright HTML test report
pnpm run test:trace       # View Playwright trace (for debugging failures)
pnpm run test:codegen     # Record interactions as test code
pnpm run test:update-snapshots # Update visual snapshots

# CI Testing
pnpm run test:e2e:ci      # Run E2E with GitHub Actions reporter
pnpm run test:e2e:shard   # Run sharded tests (parallel across machines)
```

**Cross-Browser Testing:**
- All Playwright E2E tests run on **Chromium, Firefox, and WebKit** by default
- Integration tests run on **Chromium and Firefox** (WebKit optional due to longer build times)
- Component tests run on **all three browsers** for comprehensive coverage
- GitHub Actions workflow (`.github/workflows/playwright.yml`) runs full cross-browser matrix automatically

**⚠️ First-Time Setup Required:**
- Run `pnpm run test:install` **once before running any E2E tests** to install Playwright browser binaries
- This is mandatory and takes a few minutes to complete
- After installation, all E2E, integration, and component tests will work across all browsers

**Test Organization:**
- **E2E Tests** (`tests/e2e/`): Test complete user workflows in demo site
- **Integration Tests** (`tests/integration/`): Validate built package works in real apps
- **Component Tests** (`tests/components/`): Test individual components in isolation
- **Test Fixtures** (`tests/fixtures/`): Shared helpers and utilities
- **Test Coverage**: Accessibility, keyboard navigation, nested timelines, media support, performance

### Code Quality
```bash
pnpm run eslint           # Lint TypeScript files
pnpm run fix-js           # Auto-fix ESLint issues
pnpm run format           # Format code with Prettier
pnpm run lint             # Check Prettier formatting
pnpm run lint:all         # Run all linting (ESLint + CSS + Prettier)
pnpm run clean            # Format and lint everything
pnpm run find-bugs        # TypeScript check + lint + test
pnpm run build:analyze    # Analyze bundle size (verify < 250 KB limit)

# Unit Test Debugging
pnpm run test-ui          # Interactive Vitest UI (watch mode with visual browser)
pnpm test -- --reporter=verbose  # Run tests with detailed output
```

## Architecture & Key Patterns

### 1. Styling System: Vanilla Extract Migration

The project is transitioning from styled-components to Vanilla Extract CSS-in-TypeScript for better performance and type safety.

**Style File Patterns:**
- `*.css.ts` files contain Vanilla Extract styles (modern approach)
- Imports use `import { className } from './file.css'` (note `.css` extension, not `.css.ts`)
- Components use Vanilla Extract recipes for variants and compound styles
- Legacy styled-components still exist in some files but should be avoided in new code

**When writing/editing styles:**
- Use Vanilla Extract for all new components
- Leverage the sprinkles utility system in `src/styles/system/sprinkles.css.ts`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [prabhuignoto/react-chrono](https://github.com/prabhuignoto/react-chrono) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
