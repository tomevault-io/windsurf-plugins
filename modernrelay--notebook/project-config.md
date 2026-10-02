---
trigger: always_on
description: This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## What it's for

Turn an OmniGraph graph database into a **read-and-act dashboard you describe in one YAML file** — rendered identically in a terminal and a browser.

Normally you inspect a graph by writing queries and reading JSON, or by building a bespoke UI. A notebook is the layer between: a YAML file that *declares what slices of the graph to show and what actions to allow*, not code. Each data cell is a typed lens (`Table`/`Path`/`Subgraph`/`ActionList`/`Timeline`/`Card`/`Quote`/`Text`) fed by `query.ref` to a server-owned `.gq` catalog query, or a control (`Select`/`Toggle`/`Button`) that filters state or dispatches actions. See `examples/company-server.notebook.yaml`: a clause review list with inline Approve/Reject buttons, a decisions table, and a `Signal → Decision` path — no UI code anywhere.

Two bets make it work:

- **Typed lenses, not a generic graph viewer** — you name the view you want; the system renders it.
- **Write once, render anywhere** — the same YAML drives the Ink terminal UI and the React web UI, against a live omnigraph-server (a local cluster in dev, a remote server in prod). It's bidirectional: lenses read the graph, controls and actions write back to it.

## Commands

This is a pnpm workspace (pnpm 10.30.3, Node ≥20). All scripts run from the repo root unless noted.

```bash
pnpm install                                     # install workspace deps
pnpm -r build                                    # tsc-build every package — REQUIRED before tui/web run
pnpm -r typecheck                                # tsc --noEmit across all packages
pnpm -r test                                     # vitest run across all packages

pnpm --filter @modernrelay/notebook-<pkg> build             # rebuild one package
pnpm --filter @modernrelay/notebook-<pkg> test              # vitest run for one package
pnpm --filter @modernrelay/notebook-<pkg> test -- <pattern> # single test file/name

pnpm tui examples/company-server.notebook.yaml   # Ink TUI, server mode — server URL + graph id
                                                 #   come from the notebook (run server-demo.sh first)
pnpm --filter @modernrelay/notebook-web dev                 # Vite dev server at 127.0.0.1:5173
                                                 #   add ?server=/og&graph=company (same-origin proxy)
pnpm --filter @modernrelay/notebook-web build               # tsc + vite production build

scripts/server-demo.sh                           # build omnigraph v0.7.0 CLI/server, boot a local
                                                 #   filesystem cluster (graph "company") on :8080
```

The TUI consumes built `dist/` from sibling workspace packages — **always run `pnpm -r build` after editing a non-TUI/non-web package** before running `pnpm tui`. Web's Vite bundles via TS sources directly, but `tsc --noEmit` (`pnpm -r typecheck`) is what enforces cross-package types.

## Architecture

**One catalog, two renderers, one server-backed runtime.** A notebook is YAML; each cell renders as a typed lens (`Table`/`Path`/`Subgraph`/`ActionList`) or a control (`Button`/`Toggle`/`Select`). Both the Ink TUI and the React Web app share the same catalog of component definitions and the same runtime; only the leaf component implementations and the host shell differ.

### Package map

| Package | Role |
|---|---|
| `@modernrelay/notebook-core` | The engine — start here. One package, three internal modules: `spec` (Zod schemas + YAML parser, query model — `ref`/`rawGq` — mutation specs), `catalog` (component+action definitions shared by both renderers; `assembleLensSpec` / `assembleControlSpec` produce json-render specs), `runtime` (capability-aware execution, state mirror, dependency invalidation, action dispatch, mutation lifecycle, optimistic reconciliation). The `@json-render/core` analog. |
| `@modernrelay/notebook-client` | **The only data source.** `ServerSource` + a `Client` facade over the `@modernrelay/omnigraph` SDK (`/queries/{name}` + `/query` escape hatch + `/mutate`, graph-scoped). `translateMutation` exists only for the interim `set_field` write path. |
| `@modernrelay/notebook-tui` | Ink renderer + CLI entry (`bin/omnigraph-tui.js`); host shell for terminal. |
| `@modernrelay/notebook-web` | Vite + React + Tailwind renderer; host shell for browser. |
| `@modernrelay/notebook` (`packages/cli`) | The published front-door CLI. Bundles every `@modernrelay/notebook-*` lib (tsup, `noExternal`) and ships the built web SPA in `web-dist/`. Subcommands: `view` (browser — static server + `/og` BFF proxy with server-side token injection, reusing `web/src/config.ts`'s URL-param contract), `tui` (calls `@modernrelay/notebook-tui` `main`), `validate`/`render`/`catalog`/`schema` (agent-DX, JSON out; schema via Zod 4 `z.toJSONSchema`). The workspace root is the private `notebook-workspace`; `@modernrelay/notebook` is the CLI, not the root. |

### Data flow per render

```
 YAML ─parseNotebook→ Notebook ─createNotebookRuntime→ RuntimeSnapshot ─Renderer→ UI
                         │                            │
                         ▼                            ▼

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ModernRelay/notebook](https://github.com/ModernRelay/notebook) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
