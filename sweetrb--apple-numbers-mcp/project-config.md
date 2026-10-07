---
trigger: always_on
description: This file provides guidance for AI agents (Claude, etc.) when using this MCP server.
---

# CLAUDE.md - Apple Numbers MCP Server

This file provides guidance for AI agents (Claude, etc.) when using this MCP server.

## Overview

This MCP server lets AI assistants **read, write, search, format, and import**
Apple Numbers (`.numbers`) spreadsheets. All operations are **local** — nothing
leaves the user's machine. It has two backends, and the split matters:

- **Reads *and* value/structure writes** use
  [numbers-parser](https://pypi.org/project/numbers-parser/) (Python). They open
  the `.numbers` file **directly off disk**, **offline**, with **no app and no
  special permission**. This covers `set-cell`, `add-rows`, `create-spreadsheet`,
  `import-csv` and friends — not just reads.
- **Formulas & formatting** drive **Numbers.app via AppleScript** (numbers-parser
  can't write styles/formulas reliably). Only these require **Numbers.app
  installed + the Automation permission**: `set-formula`, `set-formulas-batch`,
  `set-cell-style`, `set-cells-style-batch`, `set-column-width`, `set-row-height`,
  `merge-cells`, `unmerge-cells`.

Internalize that the split is **values vs. formatting**, not read vs. write: you
can inspect, search, export **and edit** a file with zero setup beyond the Python
sidecar. Never gate or refuse a `set-cell`, `add-rows` or `import-csv` on a missing
Numbers.app or Automation grant — those calls never send an Apple event, so a
failure there is something else. (The npm package is still macOS-only: it declares
`os: ["darwin"]`.)

## Related Documentation

- **[docs/AUTOMATION-PERMISSION.md](./docs/AUTOMATION-PERMISSION.md)** — which tools
  need Automation permission, how the prompt appears, how to grant/reset it
  (`tccutil reset AppleEvents`), what failure looks like, and how to verify.
  Required reading when a formula/format tool fails with *"Not authorized to send
  Apple events to Numbers."*
- **[docs/LIMITATIONS.md](./docs/LIMITATIONS.md)** — what the server can and can't
  do (the values-vs-formatting backend split, formulas/styles are
  AppleScript-only, `import-csv` auto-typing, fixed per-call timeouts, no
  charts/conditional-formatting, indexing, dates, format lag, concurrent edits).

## First-run requirements

1. **Python read sidecar installed.** All reads shell out to numbers-parser
   (Python). On a source clone, run `pnpm run setup` to create the project-local
   venv at `./venv` with numbers-parser. On a global install,
   `pip3 install numbers-parser`. Without it, **every read** fails.
2. **For formulas & formatting only: Numbers.app + Automation permission.** Those
   eight tools script Numbers.app, so the app must be installed and the **host
   app** (Claude, Terminal, iTerm, VS Code) must have Automation permission to
   control Numbers — granted via a one-time prompt on the first such call, or
   manually in **System Settings → Privacy & Security → Automation**. See
   [docs/AUTOMATION-PERMISSION.md](./docs/AUTOMATION-PERMISSION.md). Value and
   structure writes need neither.

When in doubt, run **`doctor`** — it's the richest diagnostic, reporting four
checks as ok/warn/fail: `python_interpreter` (the resolved Python's path and
version; warns below 3.11, since stock macOS ships 3.9 — the most common setup
failure), `numbers_parser` (read/write sidecar present), `numbers_app`
(Numbers.app present, needed for the formula/format tools), and
`automation_permission` (informational reminder that those tools need it).
`health-check` is the lighter check (numbers-parser version only).

## Conventions and behaviors to know

- **`path` is a file path with `~` expansion.** Every tool takes an explicit
  `path` (the `.numbers` file). A leading `~` is expanded to the home directory, so
  `~/Documents/budget.numbers` works. Files must end in `.numbers`.
- **File paths are bounded to the user's own space.** The tools that read or
  write ordinary files — `export-table` (`outputPath`), `create-spreadsheet`
  (`path`) and `import-csv` (`inputPath` + `outputPath`) — require the path to
  resolve, after `~` expansion **and symlink resolution**, to a location under
  the **home directory**, `/tmp`, `/private/tmp`, or `/Volumes`. Anything else
  (`/etc`, `/Library`, an app bundle, a LaunchAgents directory, or a symlink
  under `/tmp` pointing at one) is rejected with an error naming those roots.
  There is no override setting — pick a destination under one of them rather
  than retrying. Overwrite behaviour is unchanged: within the allowed roots an
  existing file at the target path is still overwritten, so confirm the
  destination.
- **Sheet/table default to the first.** Most read and value-write tools accept
  optional `sheet` / `table`; omit them to target the **first sheet** and **first
  table**. The AppleScript tools (`set-formula`, `set-cell-style`, dimensions,
  merge) generally **require** explicit `sheet` and `table`. When unsure of names,
  call `get-file-info` first.
- **Indexing is 0-based, ranges inclusive.** Every `row`/`col` (and
  `startRow`/`endRow`/`startCol`/`endCol`) is **0-based**. The **header is row 0**,
  so the first data row is **row 1** (which is why `read-table` defaults `startRow`
  to `1`). A "row 5" a human sees in Numbers.app is index `4` here. Ranges include
  both ends.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sweetrb/apple-numbers-mcp](https://github.com/sweetrb/apple-numbers-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-07 -->
