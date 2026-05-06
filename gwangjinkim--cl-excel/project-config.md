---
trigger: always_on
description: Build `cl-excel`, a Common Lisp package that reads/writes Excel **.xlsx** files with feature parity to **Julia XLSX.jl** (as documented in its Tutorial + API Reference).
---

# AGENTS.md — cl-excel (Common Lisp XLSX reader/writer)

## Mission
Build `cl-excel`, a Common Lisp package that reads/writes Excel **.xlsx** files with feature parity to **Julia XLSX.jl** (as documented in its Tutorial + API Reference).

Parity means:
- Same core operations (open/read/write/edit, sheet navigation, cell/range access, iterators, table read/write helpers).
- Same *data-type semantics* (typed values inferred from cell content + style where applicable).
- Similar limitations: “edit mode” (`rw`) is best-effort and may drop unknown OOXML parts; safe for simple data, risky around complex features (e.g., charts/formulas).

Non-goals:
- Legacy `.xls` support.
- Full-fidelity preservation of every OOXML part during `rw` (charts, drawings, pivot caches, etc.) unless explicitly required for parity.
- Full Excel calculation engine. We may preserve formulas as strings; cached results can be read when present.

Primary user experience goal:
> “I can read and write Excel files in Common Lisp as seamlessly as XLSX.jl does in Julia.”

---

## Product scope: XLSX.jl parity checklist

### 1) File open/read/write
Implement equivalents of:
- `readxlsx(source::path|io) -> workbook` (eager load)
- `openxlsx(source; mode="r"|"w"|"rw", enable_cache=true|false)` with:
  - **do-syntax equivalent** via a macro: `(with-xlsx (wb path :mode :r :enable-cache t) ...)`
  - non-macro API for manual close if desired
- `writexlsx(output, workbook; overwrite=false)` (write to path or stream)

Modes:
- `:r`  read
- `:w`  create new, blank workbook
- `:rw` edit existing (best-effort; warn loudly in docs)

Caching:
- `enable_cache=true`: cache read cells
- `enable_cache=false`: always read from disk/streaming parse (for huge sheets)

### 2) Workbook & sheet navigation
- `sheetnames(workbook) -> list-of-strings`
- `sheetcount(workbook) -> integer`
- `hassheet(workbook, name) -> boolean`
- Sheet access:
  - by name: `sheet(workbook, "mysheet")`
  - by 1-based index: `sheet(workbook, 1)`
- Sheet creation + rename for writing:
  - `addsheet!(workbook, "name") -> sheet`
  - `rename!(sheet, "new_name")`

### 3) Cell/range addressing (string references)
Support the same addressing ergonomics:
- single cell: `"B2"`
- rectangular range: `"A2:B4"`
- entire used range: `:all` (like `sheet[:]` in Julia)
- column ranges: `"A:B"`
- sheet-qualified: `"mysheet!A2:B4"`
- named ranges: `"NAMED_CELL"` at workbook scope

### 4) Reading values vs reading cell objects
Two layers:
- **Value layer**:
  - `getdata(sheet, ref)` returns scalar or 2D matrix of values
  - `readdata(source, sheet, ref)` convenience that opens and reads
- **Cell object layer**:
  - `getcell(sheet, "A1") -> cell-object` (EmptyCell if missing)
  - `getcellrange(sheet, "A1:B2") -> 2D matrix of cell-objects`

### 5) Iterators (streaming-friendly)
- `eachrow(sheet)` -> iterator yielding `SheetRow`
  - row number: `row-number(sheetrow)`
  - column access: `sheetrow[1]` or `sheetrow["B"]` or `getcell(sheetrow, 2)`
- Table iterators + helpers:
  - `eachtablerow(sheet, &key columns first-row column-labels header stop-in-empty-row stop-in-row-function keep-empty-rows)`
  - `gettable(sheet, &key ...) -> DataTable`
  - `readtable(source, sheet, &key ...) -> DataTable`
- Required `readtable/gettable` options parity:
  - `columns` as `"B:E"` or inferred contiguous block
  - `first_row` (first non-empty if omitted)
  - `header` default true
  - `column_labels` override header labels
  - `infer_eltypes` (typed columns vs Any)
  - `stop_in_empty_row` default true
  - `stop_in_row_function` predicate to end table
  - `keep_empty_rows` keep/drop rows of all missing

### 6) Supported value types (XLSX.jl parity)
XLSX.jl supports:
- String
- Missing
- Float64
- Int
- Bool
- Date
- Time
- DateTime

In Common Lisp, define a concrete, portable mapping:
- String => `string`
- Missing => `cl-excel:+missing+` sentinel (NOT `NIL`)
- Float64 => `double-float`
- Int => `integer`
- Bool => `t` or `nil` (but note: nil conflicts with “missing”, hence sentinel)
- Date/Time/DateTime => use `local-time` or a small internal struct suite:
  - `date` (Y-M-D)
  - `time` (H:M:S[.n])
  - `datetime` (date+time)
Decision: default to `local-time` if available; else provide internal structs and conversion functions.

Write-time conversions:
- Abstract numeric => coerce to integer or double-float
- `nil` values in user input => convert to `+missing+` (matches “Nothing -> Missing” behavior)
- Provide `(missing-p x)` predicate and `(maybe-value x)` helper.

Type inference:
- Use **cell style** to infer:
  - date stored as number + date format => return Date/DateTime
  - numeric visualized with decimals => return float even if stored as integer
- If cell is empty => return `+missing+`
- If cell contains empty string => return `+missing+`

---

## Architecture (how to implement XLSX safely)

XLSX is a zip of XML parts (OOXML / ECMA-376). Implement as:
1) ZIP container reader
2) Relationship resolver (`.rels`)
3) Workbook model builder:
   - workbook.xml
   - sharedStrings.xml
   - styles.xml (for number formats + date detection)
   - worksheets/sheetN.xml
   - definedNames (named ranges)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gwangjinkim/cl-excel](https://github.com/gwangjinkim/cl-excel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
