---
trigger: always_on
description: Manages exercise type definitions (Strength, Cardio, Flexibility, custom types) with field definitions. Cached with `clearCache()` method.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Principles

- **Follow established patterns** — use "Key Development Patterns" section before implementing features
- **Delegate to tools** — use build system, Jest, ESLint; don't manually validate what tools can check
- **Use services, not ad-hoc logic** — e.g., use `DataService` for CSV operations, not inline parsing
- **Update this file** — when discovering new patterns or solving complex problems, document them here

## Development Commands

```bash
npm run dev          # Development build with watch mode (CSS + esbuild)
npm run build        # Production build (tsc check + CSS + minified bundle)
npm test             # Run Jest test suite
npm run test:watch   # Jest in watch mode
npm run test:coverage # Jest with coverage report
npm run lint         # ESLint
npm run lint:fix     # ESLint with auto-fix
npm run version      # Bump version in manifest.json and versions.json
```

**Run single test**: `npm test -- app/utils/__tests__/DateUtils.test.ts`

## Build System

1. **CSS**: `node build-css.mjs` - PostCSS bundles `styles/` → `styles.css`
2. **TypeScript**: `tsc -noEmit -skipLibCheck` - Type checking only (no emit)
3. **Bundle**: esbuild bundles `main.ts` → `main.js` with Obsidian externals

The build process is sequential and must complete in order. Development mode (`npm run dev`) watches for changes and rebuilds automatically.

## Project Architecture

**Plugin Type**: Obsidian plugin for workout tracking with CSV data storage, visualizations (charts, tables, dashboards), timers, and exercise management.

**Core Principles:**

- **Service Layer Pattern**: Main plugin delegates to specialized services
- **Facade Pattern**: Services expose clean APIs while delegating to internal components
- **Atomic Design**: UI components organized by complexity (atoms → molecules → features)
- **Domain-Driven Features**: Features organized by domain (charts, tables, dashboard, modals, etc.)
- **Embedded Views**: Code blocks (`workout-chart`, `workout-log`, `workout-timer`, `workout-dashboard`) render inside notes
- **Type Safety**: Strict TypeScript with path aliases (`@app/*`)

### TypeScript Configuration

```json
{
  "baseUrl": ".",
  "noImplicitAny": true,
  "paths": { "@app/*": ["app/*"] },
  "strict": true,
  "strictNullChecks": true
}
```

**Always use `@app/*` imports instead of relative paths** (e.g., `@app/components/atoms` not `../../components/atoms`)

### Main Plugin (main.ts)

```
WorkoutChartsPlugin
├── Services
│   ├── DataService              # Facade for CSV operations (cache, columns, repository)
│   ├── ExerciseDefinitionService # Exercise type definitions and field management
│   ├── MuscleTagService          # Custom muscle tag mappings (CSV-backed, cached)
│   ├── CommandHandlerService     # Registers Obsidian commands
│   └── CodeBlockProcessorService # Registers code block processors
│
├── Embedded Views
│   ├── EmbeddedChartView         # workout-chart (Chart.js visualizations)
│   ├── EmbeddedTableView         # workout-log (sortable data tables)
│   ├── EmbeddedTimerView         # workout-timer (countdown/interval)
│   └── EmbeddedDashboardView     # workout-dashboard (stats, analytics, heat maps)
│
└── Public API
    └── WorkoutPlannerAPI         # window.WorkoutPlannerAPI for Dataview integration
```

**Key Lifecycle:**

1. `onload()`: Initialize services, register processors, expose API, add ribbon icon
2. `onunload()`: Clean up timers, views, charts, services, clear caches, nullify references
3. Service cleanup order: timers → views → cache → Chart.js → service references → ribbon → API

### Service Layer Architecture

#### DataService (Facade)

Facade over specialized data services:

- **CSVCacheService**: 5-second cache for raw CSV data
- **CSVColumnService**: CSV header management (read, ensure columns exist)
- **WorkoutLogRepository**: CRUD operations on CSV file
- **DataFilter**: Multi-strategy filtering (exact, fuzzy, filename, exercise field)

```typescript
// Usage
const data = await plugin.dataService.getWorkoutLogData({ exercise: "Squat" });
await plugin.addWorkoutLogEntry({ date, exercise, reps, weight, ... });
plugin.clearLogDataCache(); // Force refresh
```

#### ExerciseDefinitionService

Manages exercise type definitions (Strength, Cardio, Flexibility, custom types) with field definitions. Cached with `clearCache()` method.

#### MuscleTagService

Manages custom muscle tag mappings (e.g., `petto` → `chest`) from CSV file. Cache is invalidated via `triggerMuscleTagRefresh()`. Call `destroy()` to clean up.

### Embedded Views (BaseView Pattern)

All embedded views extend `BaseView` for consistent error handling, loading states, and empty data handling:

```typescript
abstract class BaseView {
  protected handleError(container, error): void;
  protected handleEmptyData(
    container,
    data,
    exercise?,
    pageLink?,
  ): boolean;
  protected renderLoadingSpinner(container): HTMLElement;
}
```

**Views:**

- `EmbeddedChartView` - Chart.js visualizations (volume, weight, reps, pace, distance, duration, heart rate)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/SpatariuRares) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
