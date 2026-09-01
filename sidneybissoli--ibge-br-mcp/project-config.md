---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

An MCP (Model Context Protocol) server, published to npm as `ibge-br-mcp`, that exposes Brazilian public data from the IBGE APIs as ~21 tools over STDIO (health data, including some DataSUS-origin stats, is read via IBGE's SIDRA — the server only ever calls IBGE endpoints). Pure TypeScript, ESM, no runtime framework — just `@modelcontextprotocol/server` (MCP SDK v2) + `zod` 4. There is no database and no local state beyond an in-memory cache; every tool is a thin async function that fetches from a public REST API and formats the result as Markdown text.

## Commands

```bash
npm run build          # tsc → dist/ (required before start/inspector; bin points at dist/index.js)
npm run dev            # build + run
npm run watch          # tsc --watch
npm test               # vitest run (all tests)
npm run test:watch     # vitest watch
npm run test:coverage  # coverage report
npm run lint           # eslint src/  (must be zero warnings)
npm run lint:fix
npm run format         # prettier --write src/
npm run inspector      # @modelcontextprotocol/inspector against dist/index.js — manual tool testing
node scripts/smoke-mcp.mjs           # smoke against the hosted Worker (initialize/tools/list/calls incl. estatisticas)
node scripts/smoke-mcp.mjs --stdio   # same smoke over local STDIO (requires npm run build)
```

Run a single test file or test by name:

```bash
npx vitest run tests/validation.test.ts
npx vitest run -t "ibgeEstados"
```

Node >= 20 (SDK v2 requirement; uses the global `fetch`). Tests mock `global.fetch` — they never hit the network.

The Cloudflare Worker (`worker/`, not published to npm) is an instance of the portfolio's Fase 0 hosting template (`mcp-br-commons/templates/cloudflare-worker`) and has its own scripts, run from inside `worker/`:

```bash
cd worker && npm run dev        # wrangler dev — local HTTP transport
cd worker && npm run deploy     # wrangler deploy
cd worker && npm run typecheck  # tsc --noEmit
cd worker && npm test           # vitest — auth, rate limit, usage, status, surface
```

The Worker intentionally does **not** list `@modelcontextprotocol/server` or `zod` in its own deps — they resolve from the parent package's `node_modules` so there is a single SDK copy (avoids duplicate-instance type clashes with `registerAll`). `worker/.npmrc` pins `legacy-peer-deps=true` so npm does not install a second copy to satisfy the `agents` package's hard peer ranges.

## Architecture

**Entry point split:** `index.ts` is a thin STDIO wrapper — it passes a `createServer` factory to the SDK's `serveStdio` (which serves both the modern and the 2025-era protocol openings) and logs to stderr. `server.ts` holds `createServer()` (builds the `McpServer`, then calls `registerAll(server)`) and the exported `registerAll(server, record?)`, which registers every tool, resource, and prompt onto a given server; the optional `record` hook (`ToolUsageRecorder`) receives `tool_call`/`tool_error` events per tool — the Worker feeds it into its usage Durable Object, STDIO passes nothing. Both are **side-effect-free and testable** (see `tests/server.test.ts`, which drives the server over the SDK's in-memory transport with the `@modelcontextprotocol/client` test client). All tool registrations and their English descriptions live in `registerAll`. The `worker/` directory (not published to npm) is a Cloudflare Worker — an instance of the Fase 0 hosting template — that serves the same `registerAll` surface over Streamable HTTP (`createMcpHandler` from `agents`, stateless, plus a `UsageTracker` Durable Object for usage stats, per-IP rate limit, optional Bearer auth, and landing/health/status/metrics routes); STDIO remains the default transport.

**Request flow for every tool:** `server.ts` registers the tool → handler calls the tool's `ibgeXxx(args)` function → that function wraps its body in `withMetrics(...)` → calls `cachedFetch(url, key, ttl)` → `cachedFetch` checks the in-memory cache, and on a miss calls `fetchWithRetry` (exponential backoff on network errors + 429/5xx) → on error the tool catches and returns `parseHttpError(...)`.

**All 21 tools are annotated read-only** via a shared `READ_ONLY` `ToolAnnotations` const in `server.ts` (`readOnlyHint`/`idempotentHint`/`openWorldHint` true, `destructiveHint` false) — every tool is a pure GET against a public API. Reference catalogs (UF/region codes, SIDRA territorial levels & table codes, biomes) are exposed as `ibge://catalogos/...` **resources** (`resources.ts`), and analysis templates (compare municipalities, demographic profile) as **prompts** (`prompts.ts`).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SidneyBissoli/ibge-br-mcp](https://github.com/SidneyBissoli/ibge-br-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-01 -->
