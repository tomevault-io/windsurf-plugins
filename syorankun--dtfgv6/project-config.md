---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

DJ DataForge v6 is an Excel-like data grid with a formula engine, plugin system, and multi-company support. It runs entirely client-side using TypeScript, Vite, and IndexedDB for persistence.

## Development Commands

### Core Development
```bash
npm run dev              # Start dev server (http://localhost:5173)
npm run build            # TypeScript compile + production build
npm run preview          # Preview production build (http://localhost:4173)
npm run type-check       # TypeScript type checking (no emit)
```

### Code Quality
```bash
npm run lint             # ESLint (src/**/*.ts)
npm run format           # Prettier formatting
```

### Testing
```bash
npm run test             # Run all tests (vitest)
npm run test:unit        # Unit tests only
npm run test:e2e         # Playwright E2E tests
npm run test:watch       # Watch mode
npm run coverage         # Coverage report
npm run bench            # Performance benchmarks
```

### Plugin Builds
```bash
npm run build:plugin     # Build plugin using vite.config.plugin.ts
npm run build:standalone # Build standalone version (pre + post processing)
```

### Important Notes About Builds
- **NEVER** open `dist/index.html` directly in browser (CORS policy blocks `file://` protocol)
- **ALWAYS** use `npm run preview` or a local HTTP server to test builds
- Production deployments should serve files from dist/ via HTTP server
- **Node.js version:** Requires Node.js >= 18.0.0 (specified in package.json)

## Architecture Overview

### Core Layers
```
┌─────────────────────────────────────────────────────┐
│              UI Layer (ui-manager.ts)               │
│  Grid View | Dashboard Mode | Selection | Toolbar  │
├─────────────────────────────────────────────────────┤
│                 Plugin Layer                        │
│  FX-Finance | Charts | ProLease | Custom Plugins   │
├─────────────────────────────────────────────────────┤
│          API Layer (PluginContext + EventBus)       │
│  Events | UI Integration | Storage API | Formulas  │
├─────────────────────────────────────────────────────┤
│                Core Services (Kernel)               │
│  Workbook | CalcEngine | Dashboard | Table | Grid  │
├─────────────────────────────────────────────────────┤
│        Persistence Layer (storage-utils)            │
│  IndexedDB | Companies | Workbooks | Plugin Data   │
└─────────────────────────────────────────────────────┘
```

### Kernel as Central Orchestrator

The **Kernel** is a singleton that orchestrates all system components:

```typescript
kernel.workbookManager   // Manages workbooks and sheets
kernel.calcEngine        // Formula parser and evaluator
kernel.storageManager    // IndexedDB persistence
kernel.companyManager    // Multi-company contexts
kernel.pluginHost        // Plugin loading and management
kernel.eventBus          // Pub/sub event system
kernel.sessionManager    // Session tracking
kernel.dashboardManager  // Dashboard creation and management
kernel.tableManager      // Table operations and data structures
```

Access kernel via: `import { kernel } from '@core/kernel'` or `DJDataForgeKernel.getInstance()`

### File Organization

The codebase uses **consolidated files** (not yet split into modules):

**Core Layer** (`src/@core/`):
- `types/index.ts` - All TypeScript types
- `workbook-consolidated.ts` - Workbook, Sheet, Column, Cell, WorkbookManager
- `calc-engine-consolidated.ts` - Parser, Evaluator, Registry, DAG
- `calc/parser.ts` - Tokenizer and AST builder (modular)
- `storage-utils-consolidated.ts` - Storage, Logger, Formatters, Assert
- `kernel.ts` - Kernel orchestrator, EventBus, CompanyManager
- `plugin-system-consolidated.ts` - Plugin host and interfaces
- `grid-virtual-consolidated.ts` - Virtual grid rendering
- `io-transform-consolidated.ts` - Import/export, transformations
- `dashboard-manager.ts` - Dashboard orchestration and state management
- `dashboard-widgets.ts` - Widget definitions and lifecycle
- `dashboard-renderer.ts` - Canvas/SVG rendering engine
- `widget-registry.ts` - Widget catalog and registration
- `table-manager.ts` - Table data structures and operations

**UI Layer**:
- `src/app.ts` - Application entry point and initialization
- `src/main.ts` - Vite entry point
- `src/ui-manager.ts` - UI state management and event handling

**Plugin Layer** (`src/plugins/`):
- `fx-finance-plugin.ts` - FX rates and financial formulas
- `charts-plugin.ts` - Chart.js integration
- `prolease-ifrs16-plugin.ts` - IFRS 16 lease accounting
- `chart-manager.ts` - Chart configuration manager
- `index.ts` - Plugin registry and loader

**NOTE**: Consolidated files have `// FUTURE SPLIT POINTS:` comments indicating where to split when refactoring.

### Path Aliases

TypeScript path mapping (configured in tsconfig.json and vite.config.ts):

```typescript
@core/*    → src/@core/*
@ui/*      → src/@ui/*
@plugins/* → src/@plugins/*
```

Always use path aliases when importing core modules.

## Working with the Codebase

### TypeScript Configuration

This project uses **strict mode** with aggressive linting:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/syorankun) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
