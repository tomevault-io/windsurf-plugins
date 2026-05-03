---
trigger: always_on
description: **headless-spreadjs** is a headless Excel workbook engine for Node.js powered by [SpreadJS](https://developer.mescius.com/spreadjs) with DOM shims. It provides a programmatic SDK and a CLI (`hsx`) for reading, writing, and manipulating `.xlsx`/`.xlsm` files without a browser or Excel. Published as `@hewliyang/headless-spreadjs` on npm.
---

# AGENTS.md

## Project Overview

**headless-spreadjs** is a headless Excel workbook engine for Node.js powered by [SpreadJS](https://developer.mescius.com/spreadjs) with DOM shims. It provides a programmatic SDK and a CLI (`hsx`) for reading, writing, and manipulating `.xlsx`/`.xlsm` files without a browser or Excel. Published as `@hewliyang/headless-spreadjs` on npm.

## Tech Stack

- **Language**: TypeScript
- **Runtime**: Node.js ≥ 18
- **Build**: `tsc` (pure TypeScript compilation)
- **Test**: Vitest
- **Linter/Formatter**: Biome
- **Excel Engine**: SpreadJS (`@mescius/spread-sheets` + optional addons for charts, pivots, shapes, slicers)
- **DOM Shim**: `happy-dom` (provides browser globals for SpreadJS in Node)
- **Canvas**: `node-canvas` (Cairo-based, for rendering/screenshots)
- **Compression**: `fflate` (for xlsx I/O)
- **Optional Peer**: Playwright (for screenshot command)

## Project Structure

```
headless-spreadjs/
├── src/
│   ├── index.ts                  # SDK entry — init(), dispose(), exports
│   ├── excel-file.ts             # ExcelFile class (open, save, toJSON, batch)
│   ├── shims.ts                  # DOM shim install/dispose (happy-dom + canvas)
│   ├── types.ts                  # GCNamespace, SpreadWorkbook, SpreadWorksheet, etc.
│   ├── vendor.d.ts               # SpreadJS vendor type declarations
│   ├── spreadjs-patches.d.ts     # SpreadJS type patches
│   └── cli/
│       ├── index.ts              # CLI entry point (bin: hsx)
│       ├── main.ts               # Argument parsing, command dispatch
│       ├── dispatch.ts           # Command router
│       ├── context.ts            # CLI context (init, file open)
│       ├── a1.ts                 # A1-style range parser
│       ├── output.ts             # Table/output formatting
│       ├── styles.ts             # Cell style utilities
│       ├── daemon.ts             # Background daemon (avoid re-init)
│       ├── daemon-entry.ts       # Daemon process entry
│       ├── client.ts             # Daemon client
│       ├── file-cache.ts         # LRU workbook cache for daemon
│       ├── abort.ts              # Abort/signal handling
│       └── commands/
│           ├── get.ts            # Read cell ranges
│           ├── set.ts            # Write cell data
│           ├── clear.ts          # Clear cells
│           ├── copy.ts           # Copy ranges
│           ├── create.ts         # Create new workbooks
│           ├── csv.ts            # CSV export (value/formula/both)
│           ├── deps.ts           # Formula dependency graph
│           ├── diff.ts           # Workbook diff
│           ├── eval.ts           # Evaluate JS against workbook
│           ├── info.ts           # Workbook/sheet info
│           ├── objects.ts        # List charts/tables/pivots
│           ├── resize.ts         # Resize ranges
│           ├── rows-cols.ts      # Row/column operations
│           ├── screenshot.ts     # Sheet screenshot (via Playwright)
│           ├── search.ts         # Search sheet data
│           └── sheet.ts          # Sheet management
├── test/
│   ├── cli.test.ts               # CLI integration tests
│   ├── daemon.test.ts            # Daemon tests
│   ├── lifecycle.test.ts         # init/dispose lifecycle tests
│   ├── workbook-core.test.ts     # Core workbook operations
│   ├── extensions/               # Extension-specific tests
│   ├── helpers.ts                # Test utilities
│   └── tsconfig.json             # Test-specific TS config
├── examples/
│   ├── basics.ts                 # Basic SDK usage
│   ├── charts.ts                 # Chart creation
│   ├── pivot-table.ts            # Pivot table creation
│   ├── shapes.ts                 # Shapes
│   ├── slicers.ts                # Slicers
│   ├── sparklines.ts             # Sparklines
│   ├── open-modify-save.ts       # Open, modify, save workflow
│   └── tsconfig.json             # Examples TS config
├── skills/
│   └── spreadjs/                 # AI skill definitions for LLM agents
├── CHANGELOG.md                  # Release changelog
├── biome.json                    # Biome linter/formatter config
├── tsconfig.json                 # TypeScript config
├── vitest.config.ts              # Vitest config
└── package.json
```

## Key Components

### SDK (`src/index.ts`, `src/excel-file.ts`)

- `init(options?)` — installs DOM shims, loads SpreadJS + optional addons, returns `{ GC, ExcelFile, dispose }`
- `ExcelFile` — open/save xlsx, buffer I/O, JSON roundtrip, `batch()` for deferred calc
- `dispose()` — tears down shims, resets state
- One active `init()`/`dispose()` lifecycle per process; multiple `ExcelFile` instances within

### CLI (`src/cli/`)

Binary: `hsx`. Commands: `get`, `set`, `clear`, `copy`, `create`, `csv`, `deps`, `diff`, `eval`, `info`, `objects`, `resize`, `rows-cols`, `screenshot`, `search`, `sheet`.

Uses a background daemon by default to avoid re-initializing SpreadJS per invocation. The daemon auto-starts, caches open workbooks (LRU), and auto-exits after idle timeout.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hewliyang/headless-spreadjs](https://github.com/hewliyang/headless-spreadjs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
