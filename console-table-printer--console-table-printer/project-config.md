---
trigger: always_on
description: This guide is for future agents and maintainers working in this repository. It summarizes how the project is organized, how changes should be made, and which checks matter before handing work back.
---

# AGENTS.md

This guide is for future agents and maintainers working in this repository. It summarizes how the project is organized, how changes should be made, and which checks matter before handing work back.

## Project Overview

`console-table-printer` is a TypeScript library for rendering formatted tables for console output. The published package is CommonJS and ships compiled files from `dist/`, with type declarations generated from TypeScript.

The public package entry point is `index.ts`. It exports:

- `Table`: chainable class facade for building and printing tables.
- `printTable`: direct helper that prints an array of row objects.
- `renderTable`: direct helper that renders an array of row objects to a string.
- `COLOR` and `ALIGNMENT`: public TypeScript type exports from the external table model.

The project uses Yarn, TypeScript, Jest with `ts-jest`, ESLint flat config, Prettier, and semantic-release.

## Repository Map

- `index.ts`: package entry point and public exports.
- `src/console-table-printer.ts`: public `Table` class. This is intentionally thin and delegates to `TableInternal`.
- `src/models/`: public and internal TypeScript model definitions.
  - `external-table.ts`: user-facing option types such as `ComplexOptions`, `ColumnOptionsRaw`, computed columns, filter/sort callbacks, and default column options.
  - `internal-table.ts`: normalized column/style structures used by the renderer.
  - `common.ts`: shared dictionary, row, color, alignment, and char-width types.
- `src/internalTable/`: internal table state, input conversion, preprocessing, and rendering.
  - `internal-table.ts`: mutable table state and methods for adding columns/rows.
  - `input-converter.ts`: converts raw user column options into internal columns.
  - `table-pre-processors.ts`: render-time preprocessing for computed columns, enabled/disabled columns, sorting, filtering, and column width calculation.
  - `internal-table-printer.ts`: actual rendering pipeline and public simple-table helpers.
- `src/utils/`: console width, padding, color, border, row, and column helpers.
- `test/`: broad integration, feature, snapshot, README example, infrastructure, package, and performance tests.
- `src/**/*.test.ts`: focused unit tests colocated with source modules.
- `static-resources/`: README screenshots.
- `.github/workflows/`: CI for lint/format, coverage, package packing, cross-version package tests, and release.

## Development Commands

Use Yarn for this repository.

```bash
yarn
yarn build
yarn test
yarn test:coverage
yarn lint
yarn format
```

Useful targeted checks:

```bash
yarn jest --config jestconfig.json path/to/file.test.ts
yarn jest --config jestconfig.json path/to/file.test.ts -u
yarn prettier --check "**/*.{ts,js,yml}"
npm pack
```

Notes:

- `yarn build` runs `tsc` and emits CommonJS JavaScript plus `.d.ts` files into `dist/`.
- `yarn test` uses `jestconfig.json` and discovers both `test/**/*.test.*` and colocated `src/**/*.test.ts` files.
- `yarn test:coverage` enforces global 80% thresholds for branches, functions, lines, and statements.
- CI uses Node 24 for build/test/quality jobs, then validates the packed package on Node 14, 16, 18, 20, and 22.

## Architecture Notes

The high-level flow is:

1. User code imports from `index.ts`.
2. `Table` in `src/console-table-printer.ts` delegates all stateful behavior to `TableInternal`.
3. User-facing options are normalized through helpers such as `rawColumnToInternalColumn` and `convertRawRowOptionsToStandard`.
4. Rendering calls `renderTable(table)` in `src/internalTable/internal-table-printer.ts`.
5. Before rendering, `preProcessColumns` and `preProcessRows` mutate the internal table state.
6. Rows are transformed, split to width-limited lines, padded, colored, bordered, and joined with newlines.

Important behavior:

- Columns can be provided explicitly, inferred from row keys, or added later.
- `defaultColumnOptions` applies when raw columns are converted to internal columns, including inferred columns and computed columns.
- `computedColumns` are created during render. The code guards against duplicate computed columns when rendering multiple times.
- Sorting and filtering happen during render and replace `table.rows` with the processed row array.
- Column transforms affect rendered values and column width calculation, but `renderRow` deep-clones the row so transforms do not mutate original row data.
- `shouldDisableColors` replaces the color map with `{}`, which leaves rendered text uncolored.
- `charLength` lets callers override width calculations for specific characters before falling back to `simple-wcswidth`.

## Working On Features

When adding or changing public behavior:

- Update or add user-facing types in `src/models/external-table.ts`.
- Keep `Table` as a thin facade unless the public API itself changes.
- Normalize user input in `src/internalTable/input-converter.ts` or `src/utils/table-helpers.ts`.
- Add internal state or defaults in `src/internalTable/internal-table.ts`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [console-table-printer/console-table-printer](https://github.com/console-table-printer/console-table-printer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
