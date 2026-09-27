---
trigger: always_on
description: > Context file for AI agents working in this repository.
---

# AGENTS.md

> Context file for AI agents working in this repository.

## Project Overview

Nx monorepo for `@koobiq/ag-grid-angular-theme` - an Angular theme package for AG Grid Community 34.x that integrates `@koobiq/design-tokens` and provides enhanced keyboard shortcuts.

## Getting Started

Use the Node.js version specified in [.nvmrc](.nvmrc).

Use yarn as the package manager. Install dependencies with:

```bash
yarn install
```

Setup git hooks:

```bash
yarn husky
```

## Common Commands

```bash
# Development
yarn run dev:theme                              # Serve dev app (port 4200)

# Building
yarn run build:theme                            # Build the publishable theme package
yarn run build                                  # Build all projects

# Lint (runs prettier, eslint, stylelint in parallel)
yarn run lint

# Auto-fix lint errors
yarn run lint:fix

# Unit Testing (Jest)
yarn run unit:theme                             # Run unit tests for the theme package
yarn run unit                                   # Run unit tests for all projects
npx jest <TEST_PATH_PATTERN>                    # Run specific unit tests (e.g., npx jest packages/ag-grid-angular-theme/tests/theme.ng.spec.ts)


# E2E Testing (Playwright)
yarn run e2e:setup                              # Install Playwright browsers
yarn run e2e:dev-ag-grid-angular                # Run E2E tests
npx playwright test <TEST_PATH_PATTERN>         # Run specific E2E tests (e.g., npx playwright test dev/ag-grid-angular/src/tests/theme.playwright-spec.ts)
npx playwright show-report                      # Show the last E2E test report

# Screenshots differ across operating systems — always use Docker to update snapshots:
yarn run e2e:docker                             # Run E2E tests in Docker (matches CI environment)
yarn run e2e:docker:update-snapshots            # Run E2E tests in Docker and update snapshots

# Clean
yarn run clean                                  # Reset Nx cache, clear Playwright cache, remove dist/tmp
```

## Repository Structure

This is a yarn monorepo managed with Nx.

```
packages/
└── ag-grid-angular-theme/                  # Published npm package
    ├── src/
    │   ├── theme.scss                      # AG Grid SCSS customization with @koobiq/design-tokens
    │   ├── state-store.ts                  # Generic store type shared by the state persistence directives
    │   ├── *.ng.ts                         # Theme component/directive
    │   └── module.ng.ts                    # Angular module
    ├── tests/
    │   └── *.spec.ts                       # Unit tests (jest)
    ├── _index.scss                         # SCSS entry point, forwards src/theme.scss
    └── index.ts                            # Public API exports

dev/
└── ag-grid-angular/                        # Demo app for development and E2E testing
    ├── public/olympic-winners.json         # Row data used by the demo and the E2E components
    ├── src/main.ts                         # Application entry point and /e2e/* routes (dev mode only)
    ├── src/overview.ng.ts                  # Showcase page (route /)
    └── src/tests/                          # Playwright E2E tests
        ├── __screenshots__/*.png           # E2E test screenshots
        ├── __snapshots__/*.txt             # E2E test snapshots
        ├── utils/*.ts                      # Utility functions for E2E tests
        ├── *.ng.ts                         # E2e test components
        └── *.playwright-spec.ts            # E2e test specs
```

## Architecture

### One Directive per Feature

Nearly every feature of `@koobiq/ag-grid-angular-theme` is a standalone attribute directive on the grid host (`selector: 'ag-grid-angular[kbqAgGrid...]'`) that calls `inject(AgGridAngular)` and either subscribes to grid outputs (`gridReady`, `firstDataRendered`, `cellKeyDown`, `cellClicked`, `cellMouseOver`) or sets grid options directly (`grid.theme`, `grid.tabToNextCell`, `grid.loadingOverlayComponent`). Directives compose freely on one `<ag-grid-angular>` element. `kbqAgGridTheme` is the base: it adds the `ag-theme-koobiq` host class (all package styles are scoped under it) and forces `grid.theme = 'legacy'`, because `theme.scss` uses AG Grid's legacy SCSS theming API and the AG Grid 33+ Theming API would conflict with it.

Feature families, all in `packages/ag-grid-angular-theme/src/`:

- Shortcuts (`select-rows-by-*`, `copy-by-ctrl-c`, `to-next-row-by-tab`): an `enabled` boolean input aliased to the selector, so `[kbqAgGridCopyByCtrlC]="false"` disables it.
- State persistence (`column-state`, `filter-state`, `quick-filter-state`, `external-filter-state`, `row-selection-state`, `row-focus-state`, plus the collapsed/selection stores inside `row-group`): see [State Persistence Pattern](#state-persistence-pattern).
- Host-component injection (`status-bar`, `row-actions`): the directive takes a component `Type` as input, instantiates it with `createComponent()` and an element injector that provides a `KBQ_AG_GRID_*_PARAMS` token (grid api, row node, ...), attaches it to `ApplicationRef`, and inserts its element into AG Grid's own DOM (`.ag-root-wrapper`, `.ag-row`).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [koobiq/data-grid](https://github.com/koobiq/data-grid) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
