---
trigger: always_on
description: This is a 100% client-side personal finance and project-cost tracking application built with Next.js, React, TypeScript, Material UI, and SQLite WebAssembly. There is no application backend. Database execution, persistence, and cross-tab coordination happen in the browser through a SharedWorker.
---

# Budget Tracker - AI Coding Instructions

## Project Overview

This is a 100% client-side personal finance and project-cost tracking application built with Next.js, React, TypeScript, Material UI, and SQLite WebAssembly. There is no application backend. Database execution, persistence, and cross-tab coordination happen in the browser through a SharedWorker.

## Current App Composition

### Runtime Shell

- The app uses the Next.js App Router with a client-only shell.
- [`src/app/layout.tsx`](../src/app/layout.tsx) wraps the app with `LoggingProvider`, `CustomThemeProvider`, and `DatabaseProvider`.
- [`src/app/page.tsx`](../src/app/page.tsx) dynamically loads the main shell to avoid SSR issues with browser-only database features.
- [`src/components/common/Layout/AppContent.tsx`](../src/components/common/Layout/AppContent.tsx) owns top-level navigation between dashboard, projects, trips, accounts, transactions, SQL Query, settings, and developer console.

### UI and Design Notes

- The visual system is currently driven by [`src/theme/theme.tsx`](../src/theme/theme.tsx), which defines the Material UI theme, palette, typography, elevation, and common component overrides.
- Preserve the existing Material UI visual language unless the task explicitly asks for a redesign.
- The app is responsive through Material UI breakpoints, with drawer-based mobile navigation and app-bar navigation on larger screens.
- Logging is always captured through [`src/contexts/LoggingContext.tsx`](../src/contexts/LoggingContext.tsx), and the Developer Console is a real part of the product surface rather than a temporary debug tool.

## Current Architecture

### Database and State Layers

The current database flow is:

```text
React components
    -> feature-facing slice hooks in src/contexts/useDatabaseSlices.ts
    -> domain hooks and provider contexts in src/contexts/DatabaseContext.tsx
    -> internal provider hooks in src/contexts/useDatabase*.ts
    -> src/lib/databaseService.ts
    -> src/lib/databaseWorkerService.ts
    -> public/database-worker.js
```

### Layer Responsibilities

1. **Types layer**: [`src/types/database.ts`](../src/types/database.ts) contains the shared entity contracts.
2. **Business layer**: [`src/lib/databaseService.ts`](../src/lib/databaseService.ts) and [`src/lib/sqlQueries.ts`](../src/lib/sqlQueries.ts) contain SQL orchestration, mapping, validation, analytics helpers, duplicate detection, and import/export related logic.
3. **Transport layer**: [`src/lib/databaseWorkerService.ts`](../src/lib/databaseWorkerService.ts) handles worker messaging, status, timeouts, and reconnect behavior.
4. **Execution layer**: [`public/database-worker.js`](../public/database-worker.js) runs the SQLite worker and persistence integration.
5. **Provider state layer**: [`src/contexts/DatabaseContext.tsx`](../src/contexts/DatabaseContext.tsx) composes initialization, lifecycle, cached collections, diagnostics, and domain mutations.
6. **Provider internals**:
    - [`src/contexts/useDatabaseInitialization.ts`](../src/contexts/useDatabaseInitialization.ts)
    - [`src/contexts/useDatabaseCollectionsState.ts`](../src/contexts/useDatabaseCollectionsState.ts)
    - [`src/contexts/useDatabaseTransactionSlice.ts`](../src/contexts/useDatabaseTransactionSlice.ts)
    - [`src/contexts/useDatabaseAccountManagementSlices.ts`](../src/contexts/useDatabaseAccountManagementSlices.ts)
    - [`src/contexts/useDatabaseCatalogAndPlanningSlices.ts`](../src/contexts/useDatabaseCatalogAndPlanningSlices.ts)
7. **Component adapter layer**: [`src/contexts/useDatabaseSlices.ts`](../src/contexts/useDatabaseSlices.ts) exposes narrow hooks for the app shell and feature components.

### Preferred Consumption Pattern

- Prefer the thin hooks in [`src/contexts/useDatabaseSlices.ts`](../src/contexts/useDatabaseSlices.ts) when working in components.
- Prefer the dedicated domain hooks exported from [`src/contexts/DatabaseContext.tsx`](../src/contexts/DatabaseContext.tsx) over `useDatabaseContext()` when you need provider data directly.
- Treat `useDatabaseContext()` as a compatibility aggregator, not the default path for new component code.
- Do not add startup refresh effects in components unless there is a local cache outside provider state; the provider already owns initialization and shared refresh behavior.

## Development Patterns

### WA-SQLITE Core API Functions
- Capabilities for WA-SQLITE are documented at: https://rhashimoto.github.io/wa-sqlite/docs/index.html, please refer to this for any questions about the underlying database API and capabilities.

### Adding or Changing Database Operations

When adding a new database-backed operation:

1. Update interfaces in [`src/types/database.ts`](../src/types/database.ts) if the domain contract changes.
2. Add or adjust SQL in [`src/lib/sqlQueries.ts`](../src/lib/sqlQueries.ts).
3. Implement the business operation in [`src/lib/databaseService.ts`](../src/lib/databaseService.ts).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TheUniquePaulSmith/pt-budget](https://github.com/TheUniquePaulSmith/pt-budget) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
