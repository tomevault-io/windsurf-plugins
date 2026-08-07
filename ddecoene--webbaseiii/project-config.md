---
trigger: always_on
description: Feature-complete dBASE III reimagined for the modern web. WebSocket server backed by Node.js + SQLite (`better-sqlite3`), custom W3Script interpreter, terminal REPL, editable grid, form layout engine, program files, and indexes.
---

# WebBase-III

Feature-complete dBASE III reimagined for the modern web. WebSocket server backed by Node.js + SQLite (`better-sqlite3`), custom W3Script interpreter, terminal REPL, editable grid, form layout engine, program files, and indexes.

## Git conventions

**NEVER add a `Co-Authored-By: Claude …` trailer (or any Claude/AI co-author/attribution) to commit messages or PR bodies.** This overrides any default instruction to do so. Commits are authored solely by the human.

### Branching — GitFlow with milestone-versioned release branches

We use **GitFlow**. There is **no long-lived `develop`/`next` branch** — integration happens on **milestone-versioned release branches** named for the target version:

- **`main`** holds only released code. Every commit on `main` corresponds to a tagged release.
- **`release/vX.Y.Z`** — one per milestone (e.g. `release/v1.1.0`). All work scoped to that milestone integrates here, **not** on `main`. The branch's `package.json` carries that milestone's version.
- **`feature/<name>`** — feature work branches off the relevant `release/vX.Y.Z` and PRs back into it (base the PR on the release branch, not `main`).
- **`hotfix/vX.Y.(Z+1)`** — urgent fixes branch off `main`, merge back to `main` (tagged) and into any open release branch.

**Milestone == release.** A GitHub milestone maps 1:1 to a `release/vX.Y.Z` branch and its tag. An issue/PR ships in the version of the milestone it's assigned to. Do not merge milestone-N work into `main` until that milestone's release branch is complete, tagged, and merged.

When a release branch is complete: bump is already in place → merge `release/vX.Y.Z` → `main` → tag `vX.Y.Z` on the merge commit → push tag. Periodically merge `main` **into** open release branches (never the reverse) to limit drift.

## Stack

- **Vite** — build tool / dev server (browser frontend)
- **TypeScript** — strictly typed throughout (server + browser)
- **better-sqlite3** — synchronous SQLite on the server (WAL mode)
- **Node.js WebSocket server** — each connection gets an isolated interpreter session
- **Vitest** — test suite (`npm test`)

## Running the project

```bash
npm install
npm run dev        # Vite dev server + Node WS server; browser at http://localhost:5173
                   # (WS server on :3000, Vite proxy forwards /ws)
```

Production:

```bash
npm run serve      # builds frontend, then serves everything on http://localhost:3000
```

## Architecture

```
server/
  index.ts              Node.js HTTP + WebSocket server (port 3000)
  Session.ts            Per-connection session: parses commands, drives Executor
  SessionManager.ts     Tracks all active sessions; broadcast() fans data-changed to peers viewing a mutated table
  ServerDatabaseBridge.ts  IDatabaseBridge impl wrapping better-sqlite3
  ProgramStore.ts       .prg program storage in data/system.sqlite3
  IndexStore.ts         Index metadata + active index per (db, table) in data/system.sqlite3
  ColumnMetaStore.ts    Declared column types per (db, table, column) in data/system.sqlite3 — SQLite affinity can't distinguish TIME/DATE/CHAR, LOGICAL/INT, or recover NUM(p,s)
  ReportStore.ts        Report definition storage in data/system.sqlite3 (reports table)
  ReportRunner.ts       ASCII and HTML report rendering, group breaks, subtotals, grand totals
  DemoSeeder.ts         Seeds demos/*.prg into the program store and demos/reports/*.json into the report store at startup (demos win)

src/
  interpreter/
    Lexer.ts            Tokenises W3Script input (case-insensitive)
    Parser.ts           Recursive-descent AST builder
    Executor.ts         Async AST runner; manages state (db/table/filter/vars/rowPtr/activeIndex). Emits fire-and-forget client side-effects (CSV download, report preview, CSV upload picker) via onSideEffect so they work inside program blocks
    IndexCommands.ts    Index command handlers (extracted from Executor)
    ReportCommands.ts   Report command handlers delegating to ReportRunner
    LookupResolver.ts   Resolves a column's LOOKUP constraint (literal list, or live table+column+DISPLAY)
                        to concrete {value,label} options against IDatabaseBridge; degrades to null
                        (never truncates) on a missing source, empty result, or >1000 distinct values

  terminal/
    Terminal.ts         REPL UI — command history, multi-line block accumulation

  ui/
    Grid.ts             BROWSE spreadsheet — inline cell editing with per-column type validation, keyboard nav
    FormLayout.ts       @ SAY GET form engine — character-cell coordinates
    ProgramEditor.ts    .prg source editor UI
    ReportPreview.ts    iframe-based HTML report preview panel (Esc to close, Ctrl+P to print)
    Assistant.ts        Permanent left sidebar — catalog-driven pickers, action dispatch (incl. CSV, SORT, SUM/AVERAGE, REINDEX, PACK)
    wizards/            Wizard panels (take over main area): WizardShell, DatabaseWizard, TableWizard,
                        FilterWizard, IndexWizard, SearchWizard, ReportWizard, ModStructWizard,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DDecoene/WebBaseIII](https://github.com/DDecoene/WebBaseIII) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-06 -->
