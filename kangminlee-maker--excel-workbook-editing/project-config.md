---
trigger: always_on
description: Design, edit, debug, reconcile, and validate Excel workbooks, workbook-generation scripts, and connected Google Sheets. Use when the task involves .xlsx files, Excel formulas, sheet layouts, named ranges, lookup logic, cross-sheet references, workbook templates, carryover flows, Google Sheets ranges, live spreadsheet formulas, validations, protected ranges, IMPORTRANGE, Apps Script-connected sheets, large or timeout-prone Sheets, external loading states, rollback snapshots, or spreadsheet review
---


# Spreadsheet Workbook Editing

Treat spreadsheets as calculation and review systems, not just tabular files.
For existing artifacts, preserve both the intended logic and the artifact identity:

- Excel workbooks are `.xlsx` files whose formula behavior must be validated by Excel.
- Existing Google Sheets are live, connected documents whose `spreadsheetId`, `sheetId` values, permissions, formulas, protections, Apps Script bindings, and external dependencies must not be broken by replacement-style workflows.

## Quick Start

1. Identify the artifact type: Excel workbook, workbook generator, existing Google Sheet, new standalone Google Sheet, or review package.
2. Separate raw inputs, parameters, intermediate calculations, derived outputs, and any prior-period carryovers.
3. Read [references/spreadsheet-principles.md](references/spreadsheet-principles.md) for shared spreadsheet rules.
4. Route to the right artifact-specific workflow:
   - Excel workbook or `.xlsx` generator: read [references/excel-workbook-principles.md](references/excel-workbook-principles.md).
   - Existing Google Sheet: read [references/connected-google-sheets-principles.md](references/connected-google-sheets-principles.md).
   - Review package or CLI preview: read [references/spreadsheet-review-package.md](references/spreadsheet-review-package.md).
5. Choose the lowest-risk edit surface: workbook formulas, named ranges, template layout, generator script, Google Sheets range patch, or validation automation.

## Task Routing

### Excel Workbooks

- **READ**: Before editing an unfamiliar workbook, run `scripts/inspect_workbook.py` to summarize sheets, dimensions, formulas, named ranges, merged ranges, tables, validations, and hidden structure.
- **EDIT**: Make deterministic workbook or generator changes in code, usually with `openpyxl` or the repo's workbook-generation layer.
- **VALIDATE**: Recalculate formula results with the real Microsoft Excel engine, preferably through `scripts/excel_engine_sample.py` for unattended temporary-copy validation.
- **SCAN**: After recalculation, run `scripts/formula_error_scan.py` to check formula and literal error cells such as `#REF!`, `#VALUE!`, `#N/A`, and `#NAME?`.
- **RECONCILE**: For approved-workbook comparisons, use a three-surface comparison: approved or golden workbook, authoritative code or calculation script, and newly generated workbook.

### Connected Google Sheets

- **GROUND**: Confirm the exact spreadsheet URL or id, tab names, `sheetId` values, target ranges, headers, formulas, validations, protected ranges, and named ranges before editing.
- **PRESERVE**: Do not download an existing Google Sheet to `.xlsx`, edit it locally, and upload it back as the default workflow. Existing Sheets are identity-bearing connected documents.
- **RISK SCAN**: Look for `IMPORTRANGE`, import functions, `QUERY`, `ARRAYFORMULA`, `INDIRECT`, custom functions, Apps Script signals, protected ranges, validation-backed cells, and external dashboards or automations before planning writes.
- **TIMEBOX**: For large or externally linked Sheets, plan reads, writes, retries, and post-write polling under explicit timeout and quota budgets.
- **PATCH**: Prefer narrow Google Sheets API or connector edits against existing ranges. Avoid deleting and recreating tabs, replacing whole sheets, or overwriting formulas with displayed values unless the user explicitly requests that behavior.
- **READBACK**: Re-read the changed cells, formulas, validations, import/load states, and key dependent outputs from the live Google Sheet after writing.

### Review Packages

- **PACKAGE**: When the user needs CLI-visible evidence, produce a self-contained review package with HTML previews, JSON structure, formula/dependency risk logs, key values, and before/after summaries.
- **SEPARATE**: Treat visualization as evidence, not calculation proof. Report the calculation engine or live readback used for validation.

## Editing Defaults

- Preserve a traceable path from inputs to outputs.
- Keep important parameters visible and centralized.
- When modifying an existing template, study and preserve its established formatting, sheet conventions, named-range patterns, and reviewer-facing layout unless the user explicitly asks to change them.
- Do not add a second workbook-only logic path that silently diverges from the authoritative logic.
- Do not hide known input limitations inside ad hoc override formulas.
- Prefer compatibility-safe formulas over newer functions when the target spreadsheet environment is uncertain.
- For recurring period workbooks, separate prior-period carry-ins, current-period raw inputs, and next-period carry-outs explicitly instead of mixing them into one surface.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kangminlee-maker/excel-workbook-editing](https://github.com/kangminlee-maker/excel-workbook-editing) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
