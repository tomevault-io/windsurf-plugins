---
trigger: always_on
description: This file tells coding agents (Claude Code, Cursor, etc.) how to use **Taible**
---

# AGENTS.md — Using Taible (for AI coding agents)

This file tells coding agents (Claude Code, Cursor, etc.) how to use **Taible**
correctly. It is task-oriented and copy-paste friendly. For human prose see
`docs/USAGE.md`; for the full feature log see `docs/PROGRESS.md`.

## What Taible is

A clean-room, **MIT-licensed** data grid + spreadsheet engine for **React /
Next.js**. No GPL/Handsontable/HyperFormula code. Canvas rendering + DOM editing
hybrid, a self-built Excel-compatible formula engine (150 functions), CRDT
collaboration, AI-native helpers, and an MCP tool layer.

Monorepo (pnpm). Eight published packages:

| Package | Import from | Purpose |
|---|---|---|
| `@ai-path/tb-core` | `@ai-path/tb-core` | Headless models: sizes, selection, undo, merge, validation, conditional format (value + visual), number format, aggregate, pivot, sparkline, chart layout, detail, fill, coords. No React/DOM. |
| `@ai-path/tb-formula` | `@ai-path/tb-formula` | `SheetEngine`, parser, 150 functions, dependency graph, spill, named ranges, R1C1, structured refs. |
| `@ai-path/tb-data` | `@ai-path/tb-data` | Visual↔physical index mapping, sort/filter models, nested rows, `DataView`, `AsyncRowModel`. |
| `@ai-path/tb-react` | `@ai-path/tb-react` | `<LatticaGrid>`, `<LatticaStatusBar>`, `<LatticaChart>`, `useGridController`, themes/palette/density. |
| `@ai-path/tb-io` | `@ai-path/tb-io` | CSV/TSV, XLSX read + plain/`writeStyledXlsx`, JSON, clipboard, `tableToPdf`. |
| `@ai-path/tb-collab` | `@ai-path/tb-collab` | CRDT (LWW), fractional index keys, presence, transport. |
| `@ai-path/tb-ai` | `@ai-path/tb-ai` | Provider-agnostic NL→formula/operation, smart fill, anomaly, etc. (MockProvider for tests). |
| `@ai-path/tb-mcp` | `@ai-path/tb-mcp` | Grid tool registry + `ToolDispatcher` for AI agents. |

## Install

```bash
pnpm add @ai-path/tb-react @ai-path/tb-core @ai-path/tb-formula
# add @ai-path/tb-io @ai-path/tb-data @ai-path/tb-ai @ai-path/tb-mcp @ai-path/tb-collab as needed
```

Peer deps: `react`/`react-dom` ≥ 18 (tested on 19). ESM + CJS are both shipped.

> **Import rule:** consumers import from the package name (`@ai-path/tb-react`).
> The `.js` suffix you see on *internal* relative imports (`./foo.js`) is a
> source convention for NodeNext ESM — do **not** add it to package imports.

## Quickstart — a React grid

```tsx
'use client';
import { useState } from 'react';
import { LatticaGrid, useGridController } from '@ai-path/tb-react';
import type { ColumnNode } from '@ai-path/tb-core';

const columns: ColumnNode[] = [
  { headerName: 'Item', field: 'item', width: 180, type: 'text' },
  { headerName: 'Qty', field: 'qty', width: 80, type: 'number', align: 'right' },
  { headerName: 'Total', field: 'total', width: 120, type: 'number', format: '#,##0' },
];

export default function Demo() {
  const controller = useGridController({ rowCount: 1, colCount: 1 });
  const [rows] = useState([
    { item: 'Apple', qty: 3, total: '=B1*100' },
    { item: 'Pear', qty: 2, total: '=B2*120' },
  ]);
  return <LatticaGrid controller={controller} rows={rows} columns={columns} width={800} height={480} />;
}
```

- `useGridController(options)` returns a stable headless `GridController`.
- `<LatticaGrid controller rows columns width height autoSize maxWidth maxHeight fill theme renderDetail contextMenu onCellCommit editSelection cellOverlay renderCellOverlay onCellOverlayClose />`.
- `columns` are optional multi-level header defs (`ColumnNode` = leaf `{headerName}`
  or group `{headerName, children, collapsible?, showWhen?}`). Leaf defs may carry
  rich metadata such as `field`, `width`, `type`, `editable`, `align`, `format`,
  `options`, and `maxLength`. Omit for A,B,C… letters.
- `rows` is controlled record data. Leaf `field` values define the extraction order;
  a fieldless leaf receives empty cells. When `rows` changes, the controller resizes
  and replaces grid data without adding undo history.
- The imperative API remains available for edits and programmatic writes:
  `controller.setCellText(0, 0, 'Apple')`, `controller.setData(matrix)`, and
  `controller.setRecords(records, fields)`.
- `autoSize="content"` sizes the grid to visible content
  (`rowHeaderWidth + visible column widths`, `colHeaderHeight + visible row heights`).
  `maxWidth` / `maxHeight` clamp that size and leave overflow scrollable. When
  `autoSize` is set, `width`, `height`, and `fill` are ignored.

## Public contract (stable identifiers)

The following DOM hooks are public and stable. Renaming or changing their
semantics is a breaking change:

- `data-testid="lattica-grid"`
- `data-testid="lattica-editor"`, `lattica-editor-select`,
  `lattica-editor-date`, `lattica-editor-autocomplete`, `lattica-editor-datalist`
- `data-testid="lattica-filter-<col>"`, `lattica-sort-<col>`
- `data-testid="lattica-cell-overlay"`
- `data-testid="lattica-colsettings"`, `lattica-colsettings-vis-<physicalCol>`,
  `lattica-colsettings-width-<physicalCol>`, `lattica-colsettings-showall`,
  `lattica-colsettings-resetwidths`
- `data-testid="lattica-static-table"`
- `data-testid="lattica-rowgroup-<row>"`

ARIA labels are also stable for interactive chrome:
`filter column N`, `sort column N`, `toggle row group N`, and column-settings

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aipathjp/lattica](https://github.com/aipathjp/lattica) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-30 -->
