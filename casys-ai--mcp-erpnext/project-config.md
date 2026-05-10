---
trigger: always_on
description: Repository guidelines for AI coding agents working on this codebase.
---

# AGENTS.md

Repository guidelines for AI coding agents working on this codebase.

- Repo: https://github.com/Casys-AI/mcp-erpnext
- In chat replies, file references must be repo-root relative only (example: `src/tools/sales.ts:42`); never absolute paths.

## Project Overview

MCP server for ERPNext/Frappe ERP — 120+ tools across 14 categories with 7 interactive UI viewers. Connects MCP-compatible AI agents to ERPNext via the Model Context Protocol. Published as `@casys/mcp-erpnext` on npm (Node bundle) and JSR (Deno).

## Project Structure & Module Organization

- **Entry points**: `mod.ts` (JSR public API), `server.ts` (MCP server — stdio + HTTP).
- **Config**: `deno.json` is the primary config (version, tasks, import map). `src/ui/package.json` manages UI-only npm deps (React, Vite, Recharts).
- **Source code**: all under `src/`. Server-side TypeScript uses Deno conventions; UI viewers use Vite/React with standard npm imports.
- **Tool categories**: one file per category under `src/tools/` (`sales.ts`, `inventory.ts`, `accounting.ts`, etc.). All registered in `src/tools/mod.ts`.
- **Tests**: colocated with source files (Deno convention) — `foo.ts` / `foo_test.ts`. Not all modules have tests yet; the convention is the target, not a guarantee.
- **UI viewers**: each viewer is a standalone React app under `src/ui/{viewer-name}/`, bundled to a single HTML file. Built output goes to `src/ui/dist/` (gitignored but included in published artifacts via `deno.json` publish config).
- **Kanban**: `src/kanban/` contains types, definitions, field-utils, and per-DocType adapters in `adapters/`.
- **Runtime adapters**: `src/runtime.ts` (Deno) and `src/runtime.node.ts` (Node.js) — the build script swaps them.
- **Scripts**: `scripts/build-node.sh` produces the npm bundle.
- **Docs**: `docs/` contains roadmap, known issues, and coverage notes.
- Keep UI-only deps in `src/ui/package.json`; do not add them to `deno.json`. Conversely, server-side deps go in the `deno.json` import map.

## Build, Test, and Development Commands

```bash
# Run all tests (also: deno task test)
deno test --allow-all src/

# Run a single test file
deno test --allow-all src/tools/sales_test.ts

# Type check
deno check mod.ts server.ts

# Format (Deno built-in)
deno fmt

# Lint (Deno built-in)
deno lint

# Start HTTP server (dev)
deno task serve                    # --http --port=3012

# Start with MCP inspector
deno task inspect

# Compile to standalone binary
deno task compile

# Build UI viewers
deno task ui:build                 # or: cd src/ui && npm ci && node build-all.mjs

# Build Node.js npm bundle
deno task ui:build && ./scripts/build-node.sh

# Dev a specific UI viewer with HMR
cd src/ui && npm run dev:kanban    # also: dev:invoice, dev:stock, dev:doclist
```

- Runtime baseline: **Deno 2.x** (development), **Node 20+** (npm bundle target — CI uses Node 22).
- If UI deps are missing, run `cd src/ui && npm ci` (prefer `npm ci` over `npm install` for reproducibility).
- Run `deno test --allow-all src/` before pushing when you touch logic.
- Run `deno check mod.ts server.ts` to verify type safety after changes.
- Hard gate: if the change affects the build pipeline or published surfaces, `scripts/build-node.sh` must be tested.

## Architecture

### Dual-runtime design

The project runs on **Deno** (development/JSR) and **Node.js** (npm). Platform-specific APIs are abstracted through a runtime adapter:
- `src/runtime.ts` — Deno implementation (uses `Deno.env`, `Deno.readTextFile`, etc.)
- `src/runtime.node.ts` — Node.js implementation (uses `process.env`, `node:fs`)

The build script `scripts/build-node.sh` swaps `runtime.ts` with `runtime.node.ts`, strips `.ts` extensions from imports, and produces a single esbuild bundle at `dist-node/bin/mcp-erpnext.mjs`.

**All source code imports `from "./runtime.ts"` — never import Deno or Node APIs directly.**

### Tool architecture

Each tool is an `ErpNextTool` object (`src/tools/types.ts`) with: `name`, `description`, `category`, `inputSchema` (JSON Schema), `handler`, and optional `_meta` (UI viewer binding). Tools are grouped by category in individual files under `src/tools/`, registered in `src/tools/mod.ts`, and exposed through `ErpNextToolsClient` (`src/client.ts`).

Tool naming: `erpnext_{entity}_{operation}` (e.g. `erpnext_customer_list`, `erpnext_sales_order_create`).

The `handler` receives `(input, ctx)` where `ctx.client` is the `FrappeClient` singleton. The client is lazily initialized from env vars on first use.

### Frappe REST client

`src/api/frappe-client.ts` is a zero-dependency HTTP client wrapping the Frappe REST API. Key methods: `list()`, `get()`, `create()`, `update()`, `delete()`, `callMethod()`. All errors throw `FrappeAPIError` with HTTP status and parsed body — no silent fallbacks.

**Submit handlers must GET the doc first** to pass `modified` for Frappe's optimistic locking (see `docs/known-issues.md`). Cancel does not need this.

### Kanban system

The kanban viewer is the canonical read-write MCP App. Architecture:
- `src/kanban/types.ts` — shared contracts (`KanbanBoard`, `KanbanCard`, `KanbanAdapter`, etc.)
- `src/kanban/definitions.ts` — board registry (Task, Opportunity, Issue)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Casys-AI/mcp-erpnext](https://github.com/Casys-AI/mcp-erpnext) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
