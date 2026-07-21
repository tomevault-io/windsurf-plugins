---
trigger: always_on
description: Grafana panel plugin (`briangann-datatable-panel`) that renders query results with
---

# AGENTS.md - Datatable Panel Plugin

Grafana panel plugin (`briangann-datatable-panel`) that renders query results with
[DataTables.net](http://www.datatables.net). Targets Grafana 12.3+. Frontend-only plugin (no backend / no `Magefile.go`).

Package manager is `pnpm` (see `packageManager` field in `package.json`). Node >= 24 (`.nvmrc` pins the version).

## Commands

Build, lint, test:

- `pnpm run dev` — webpack watch build (outputs `dist/`).
- `pnpm run build` — production webpack build.
- `pnpm run typecheck` — `tsc --noEmit`.
- `pnpm run lint` / `pnpm run lint:fix` — ESLint (+ Prettier on `:fix`).
- `pnpm run test` — Jest watch mode (only changed files).
- `pnpm run test:ci` — full Jest run used in CI (`--passWithNoTests --maxWorkers 4`, coverage on).
- Run a single Jest test file: `pnpm exec jest src/data/columns/columnStyles.test.ts`.
- Run a single test by name: `pnpm exec jest -t "partial test name"`.
- `pnpm run spellcheck` — cspell.

E2E (requires a running Grafana with the plugin mounted):

- `pnpm run server` — `docker compose up --build`. Launches Grafana on `:3000` with `dist/` bind-mounted at
  `/var/lib/grafana/plugins/briangann-datatable-panel`. Override Grafana version with `GRAFANA_VERSION=…` / image with
  `GRAFANA_IMAGE=…`. `dist/` must be built first (`pnpm run build` or `dev`).
- `pnpm run playwright:test` — run Playwright specs under `tests/` against `GRAFANA_URL` (default
  `http://localhost:3000`). The `auth` project logs in and caches cookies to `playwright/.auth/admin.json` before the
  `chromium` project runs.
- `pnpm run playwright:test:ui` / `pnpm run playwright:showreport` — Playwright UI / HTML report.

## Architecture

### `src/` layout

```text
src/
├── components/                  React components + Grafana panel options UI
│   ├── DataTablePanel.tsx       the panel
│   ├── styles.ts                Emotion styling
│   └── options/                 editor UI (Grafana options panel)
│       ├── columnstyles/        per-column style editor (ColumnStyleItem + ColumnStylesEditor)
│       └── thresholds/          per-style threshold editor (ThresholdItem + ThresholdsEditor)
├── data/                        panel options → DataTables config pipeline
│   ├── dataHelpers.ts           orchestrator (DataFrame → DataTables rows)
│   ├── transformations.ts       Grafana DataFrame transformers
│   ├── cells/                   cell-level rendering (formatter, createdCell, colors)
│   ├── columns/                 column-level config (alias, style matcher, width hints)
│   ├── layout/                  DataTables `layout` config (search bar position)
│   └── mappings/                Grafana value/range/regex mapping resolution
├── hooks/                       generic React hooks (useApplyTransformation, useTracker)
├── utils/                       cross-cutting helpers (color math)
├── images/, img/, screenshots/  static assets
├── migrations.ts                Angular → React options migration + defaults backfill
├── module.ts                    plugin entry (PanelPlugin registration)
├── plugin.json                  Grafana plugin manifest
└── types.ts                     single source of truth for all domain + runtime types
```

### Panel entry point & option pipeline

- `src/module.ts` registers the `PanelPlugin`, wires `DatatablePanelMigrationHandler` (for Angular → React option
  migration), disables most `FieldConfigProperty.*` standard options, and attaches `optionsBuilder` for the editor UI.
- `src/types.ts` is the source of truth for `DatatableOptions` and all option enums (transformation, paging, sorting,
  aggregations). Changes here almost always require corresponding updates to `src/migrations.ts` and
  `src/components/options/defaults.ts`.
- `src/components/options/optionsBuilder.tsx` wires editor controls (column aliases, width hints, sorting, column
  styles, thresholds) into the Grafana panel options panel. Style/threshold editors live in
  `src/components/options/columnstyles/` and `src/components/options/thresholds/`.

### Runtime data flow (panel render)

`src/components/DataTablePanel.tsx` is the single React component that renders the table. The pipeline for each render:

1. `GetDataTransformerID(props.options.transformation)` (`src/data/transformations.ts`) maps the plugin's
   `TransformationOptions` enum to a Grafana `DataTransformerID` (`joinByField`, `merge`, `reduce`, `seriesToRows`).
   `reduce` carries the user's aggregation list.
2. `useApplyTransformation` (`src/hooks/useApplyTransformation.ts`) runs the transformer over `props.data.series` via
   `transformDataFrame` + `lastValueFrom`, producing the `DataFrame[]` the table consumes.
3. `BuildColumnDefs` + `ConvertDataFrameToDataTableFormat` (`src/data/dataHelpers.ts`) turn the DataFrames into
   DataTables.net `columns` / `rows`.
4. `ApplyColumnAliases` (`data/columns/columnAliasing.ts`) and `ApplyColumnWidthHints`
   (`data/columns/columnWidthHints.ts`) mutate the column defs based on user options.
5. `data/cells/cellRenderer.ts` + `data/cells/createdCellHelpers.ts` produce per-cell `render` / `createdCell` callbacks
   that apply column styles, value/range mappings (`data/mappings/valueMappings.ts`,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [briangann/grafana-datatable-panel](https://github.com/briangann/grafana-datatable-panel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
