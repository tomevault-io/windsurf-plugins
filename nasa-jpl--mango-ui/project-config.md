---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Quick Commands

- **Development**: `npm run dev` — Start dev server on https://localhost:5173/mango
- **Build**: `npm run build` — TypeScript check + Vite build (outputs to `build/`)
- **Unit tests**: `npm run test` (watch mode) or `npm run test:unit` (single run)
- **E2E tests**: `npm run test:e2e` or `npm run test:e2e:ui` (with UI) or `npm run test:e2e:debug` (debug mode)
- **Lint**: `npm run lint` (JavaScript/TypeScript), `npm run lint:css` (CSS), `npm run lint:css:fix` (auto-fix CSS)

## Project Overview

MANGO UI is a React/TypeScript frontend for the GRACE satellite missions monitoring tool. It enables visualization and analysis of telemetry data from multiple missions, with features for data comparison, plotting, and export.

### Tech Stack
- **Framework**: React 18 + TypeScript, Vite for build/dev
- **UI**: Tailwind CSS, NASA JPL Stellar React design system, Radix UI components
- **Data visualization**: Chart.js, D3 scales/formatting, Cesium 3D maps, ag-grid
- **Testing**: Vitest (unit), Playwright (e2e)
- **Routing**: React Router v6 with loaders for data fetching

### Development Setup

The project requires:
1. **Self-signed certificates** for HTTPS dev server:
   ```bash
   brew install mkcert
   mkdir -p .cert && mkcert -key-file ./.cert/key.pem -cert-file ./.cert/cert.pem 'localhost'
   ```
2. **Environment variables** in `.env` file. Use `.env.template` as reference; contains VITE_API_URL and VITE_MANGO_DOCS_URL (contact repo owners for access to configuration repository)
3. **Node.js**: Use `nvm use` to activate required version (configured in `.nvmrc`)

### Build Pipeline

`npm run build` enforces **strict TypeScript checking before any bundling** — it runs `tsc` first, then `vite build`. Build fails if TypeScript errors exist. If you need to skip TypeScript to debug build issues, use `npm run build:force`, but this bypasses type safety.

### Directory Structure

```
src/
├── routes/           # Page-level components (HomePage, ViewPage, ManagementPage, ProductsPage)
├── components/
│   ├── app/          # Layout & shell (Sidebar, SaveViewModal, ProductTable)
│   ├── page/         # Page layout building blocks (Section, Entity, EntityHeader, CustomGridItem)
│   ├── entities/     # Data visualization components
│   │   ├── chart/    # Chart.js wrapper with tooltips
│   │   ├── map/      # Cesium 3D maps
│   │   ├── table/    # ag-grid tables with custom filtering
│   │   ├── text/     # Text display
│   │   ├── timeline/ # Timeline visualizations
│   │   └── timeline-row/ # Row-level timeline component
│   ├── mission/      # Mission-specific features (e.g., GRACE DownlinkDashboard)
│   └── ui/           # Reusable UI components (DataGrid, DateRangePicker, ProductSelector, Tabs, Tooltip, etc.)
├── utilities/
│   ├── api.ts        # API client (fetch wrapper, getView)
│   ├── view.ts       # View state management (createView, serialization)
│   ├── time.ts       # Time calculations and utilities
│   ├── product.ts    # Product data transformations
│   ├── generic.ts    # General-purpose utilities
│   ├── dataset.ts    # Dataset-specific helpers
├── types/            # TypeScript types (api, view, app, data-grid, time, status, page)
├── main.tsx          # React Router setup, global providers (TooltipProvider, AlertDialogProvider, Toaster)
├── config.ts         # App configuration
└── [index|variables].css  # Global styles
```

### Key Architectural Patterns

**Page Layout**: Pages are composed using `ViewPage` → `Section` → `Entity` pattern, where each `Entity` wraps a visualization component (Chart, Map, Table, etc.). `CustomGridItem` provides grid layout positioning.

**State & Data**: Views (custom dashboard configurations) are loaded via React Router loaders; the API client fetches and serializes view state. The app supports creating, saving, and sharing custom views via URL parameters.

**Design System**: Uses NASA JPL Stellar React for consistent theming and components. Global CSS variables defined in `variables.css` provide the design tokens.

**API Integration**: All backend calls go through `utilities/api.ts`. Development server proxies `/api/*` to the backend URL specified in `.env` (configured in `vite.config.ts`).

### Testing

- **Unit tests**: Collocated with source files (`*.test.ts`). Run via Vitest. Output to `unit-test-results/`.
- **E2E tests**: In `playwright.config.ts`. Playwright codegen available: `npm run test:e2e:codegen` (opens recording UI at https://localhost:5173).

### Building & Deployment

Production builds are static HTML/CSS/JS (Cesium assets copied separately). The `base` path is configurable via `VITE_APP_PATH` environment variable. Docker image available; runtime injects `VITE_API_URL` and `VITE_MANGO_DOCS_URL`.

### View State & Persistence

Views are the core of MANGO — they define custom dashboards and are saved to the backend. The view hierarchy is:

```
View (config: sidebar width, date range bounds 2010–2050)
├─ home: Page (single landing page)
└─ pageGroups: PageGroup[]
   └─ pages: Page[]
      └─ sections: Section[]

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nasa-jpl/mango-ui](https://github.com/nasa-jpl/mango-ui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-11 -->
