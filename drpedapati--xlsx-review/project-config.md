---
trigger: always_on
description: Use this skill when the user wants to create, edit, read, or diff Excel spreadsheets (.xlsx). Triggers: 'spreadsheet', 'Excel', '.xlsx', 'table in Excel', 'create a spreadsheet'. Do NOT use for CSV files, Google Sheets, or non-Excel tabular data.
---


# Excel (.xlsx) Creation, Editing, and Analysis

Create, edit, read, and diff Excel spreadsheets using `xlsx-review`, a CLI tool built on Microsoft's Open XML SDK. Ships as a self-contained native binary — no runtime required.

## Quick Reference

| Task | Command |
|------|---------|
| Create new workbook | `xlsx-review --create -o output.xlsx manifest.json` |
| Create empty workbook | `xlsx-review --create -o output.xlsx` |
| Create from template | `xlsx-review --create --template base.xlsx -o output.xlsx manifest.json` |
| Edit existing workbook | `xlsx-review input.xlsx edits.json -o output.xlsx` |
| Read workbook contents | `xlsx-review input.xlsx --read --json` |
| Read (human-readable) | `xlsx-review input.xlsx --read` |
| Diff two spreadsheets | `xlsx-review --diff old.xlsx new.xlsx` |
| Diff (JSON output) | `xlsx-review --diff old.xlsx new.xlsx --json` |
| Dry run (validate only) | `xlsx-review input.xlsx edits.json --dry-run` |
| Pipe JSON from stdin | `cat edits.json \| xlsx-review input.xlsx -o output.xlsx` |

## Warnings — Known Corrupt-File Bugs

**READ THESE BEFORE GENERATING ANY MANIFEST. Violating these rules produces files that open with a repair dialog or fail to open entirely.**

1. **NEVER combine `set_table` with `set_auto_filter` on the same range.** Tables include auto-filter automatically. Using both produces a corrupt file. If you need a filterable table, use `set_table` alone.

2. **NEVER use `set_page_orientation` or `set_print_area`.** These produce invalid XML element ordering. The resulting file opens with a repair dialog in Excel. Omit print layout changes entirely.

3. **NEVER use `comments` on cells that already have comments from a previous edit pass.** This produces invalid `legacyDrawing` element ordering. The file opens with a repair dialog.

4. **`--create` mode disables yellow highlighting automatically.** In edit mode, cells are highlighted yellow to show changes. In create mode (v1.3.0+), this is disabled. For edit mode without highlighting, use `--no-highlight`.

5. **Always validate after creating.** Run `xlsx-review output.xlsx --read --json` and check that the structure matches expectations. For critical files, open in Excel or LibreOffice to verify.

## Workflow

### Step 1 — Plan the spreadsheet structure

Before writing JSON, decide: sheet names, column headers, data types (text vs. number), which ranges need formulas, tables, validation, or conditional formatting. State the plan, then write the manifest.

### Step 2 — Write the JSON manifest

The manifest is a JSON object with `author` (optional), `changes` (array of operations), and `comments` (optional array). Operations execute in array order.

```json
{
  "author": "Author Name",
  "changes": [
    { "type": "...", ... }
  ],
  "comments": [
    { "sheet": "Sheet1", "cell": "A1", "text": "Note text" }
  ]
}
```

### Step 3 — Run the tool

```bash
# Create new workbook and populate it
xlsx-review --create -o output.xlsx manifest.json --json

# Edit existing workbook
xlsx-review input.xlsx edits.json -o output.xlsx --json
```

### Step 4 — Validate the output

```bash
# Read back the workbook and verify structure
xlsx-review output.xlsx --read --json
```

Check that cell values, formulas, sheet names, and table definitions match expectations. If the output JSON shows unexpected structure, fix the manifest and re-run.

## Change Type Reference

Every operation goes in the `changes` array. All fields are required unless marked optional.

### Cell Operations

**`set_cell`** — Set a cell's value.

```json
{ "type": "set_cell", "sheet": "Sheet1", "cell": "A1", "value": "Hello" }
{ "type": "set_cell", "sheet": "Sheet1", "cell": "B2", "value": "42", "format": "number" }
```

- `sheet` — worksheet name (case-sensitive)
- `cell` — A1 notation
- `value` — string value
- `format` (optional) — `"number"` to store as numeric. Without this, all values are stored as strings.

**`set_formula`** — Set a cell formula.

```json
{ "type": "set_formula", "sheet": "Sheet1", "cell": "C1", "formula": "=SUM(A1:B1)" }
{ "type": "set_formula", "sheet": "Summary", "cell": "A1", "formula": "COUNTA(Sheet1!A:A)-1" }
```

- `formula` — Excel formula. The leading `=` is optional; the tool adds it if missing.

### Row and Column Operations

**`insert_row`** — Insert a blank row after a specified row number.

```json
{ "type": "insert_row", "sheet": "Sheet1", "after": 5 }
```

**`delete_row`** — Delete a row (shifts rows up).

```json
{ "type": "delete_row", "sheet": "Sheet1", "row": 10 }
```

**`insert_column`** — Insert a blank column after a specified column letter.

```json
{ "type": "insert_column", "sheet": "Sheet1", "after": "C" }
```

**`delete_column`** — Delete a column (shifts columns left).

```json
{ "type": "delete_column", "sheet": "Sheet1", "column": "D" }
```

### Sheet Operations

**`add_sheet`** — Add a new worksheet.

```json
{ "type": "add_sheet", "name": "Summary" }
```

**`rename_sheet`** — Rename a worksheet.

```json
{ "type": "rename_sheet", "from": "Sheet1", "to": "Data" }
```

**`delete_sheet`** — Delete a worksheet.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [drpedapati/xlsx-review](https://github.com/drpedapati/xlsx-review) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
