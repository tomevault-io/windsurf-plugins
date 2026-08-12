---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

ABAP MCP Server v2 is a standalone Model Context Protocol (MCP) server that enables AI assistants (Claude, Copilot, Cursor) to interact with SAP ABAP systems via the ADT REST API. It implements 67 tools across 16 categories + 2 meta-tools (`find_tools`, `list_tools`) + 1 MCP Prompt (`abap_develop`) for full ABAP development workflow support.

## Build & Development Commands

```bash
# Install dependencies
npm install

# Build (TypeScript compilation)
npm run build

# Development mode (uses tsx for direct TypeScript execution)
npm run dev

# Start built server
npm start

# Run unit tests (Vitest — no SAP connection required)
npm test

# Run unit tests in watch mode
npm run test:watch

# Clean build artifacts
npm run clean
```

**Tech Stack:**
- TypeScript 5.7+ with strict mode
- Node.js 20+
- Target: ES2022
- Output: ESM (`"type": "module"`) to `dist/`, module resolution: NodeNext
- Tests: Vitest (`test/*.test.ts`) covering pure helpers (Clean ABAP parsing, SAProuter
  route parsing, safety guards, config parsing). Tests do **not** hit a live SAP system;
  `vitest.config.ts` injects dummy env + a `.js`→`.ts` resolver for the NodeNext imports.

## Architecture

### Modular Architecture
- **Entry**: `src/index.ts` — startup & banner
- **Server**: `src/server.ts` — MCP request handlers (ListTools, CallTool, Prompts)
- **Config**: `src/config.ts` — environment variable parsing
- **Schemas**: `src/schemas.ts` — Zod parameter validation for all tools
- **Tools**: `src/tools/` — tool definitions, registry, and handler dispatch map
  - `tool-definitions.ts` — 67 tool metadata (name, description, schema, optional `requiresAdt: false` for tools that never touch SAP)
  - `tool-registry.ts` — categories, core tools, deferred loading
  - `handler-map.ts` — dispatch map (tool name → handler function)
  - `handlers/` — 20 handler modules (search, read, write, create, delete, test, quality, diagnostics, transport, abapgit, query, documentation, context, websearch, batch, meta, method, contract, analysis, intent)
- **Helpers**: `src/helpers/` — JSON schema conversion, DDIC validation, documentation fetching, Clean ABAP analysis, method-splice (single-method surgery), contract (context compression), transport-resolve (reuse an open request instead of auto-creating new ones)
- **Cache**: `src/cache.ts` — TTL-bounded `getObjectSource` cache, invalidated on write/delete
- **Audit**: `src/audit.ts` — structured JSON audit log of write/delete/execute (to stderr + optional file). write/edit/delete handlers audit internally; all other mutating tools (creates, activates, abapGit pull, transport, snippet outcome) are covered by the `withAudit` wrapper in `handler-map.ts`, driven by `AUDIT_WRAPPED_TOOLS` in `tools/mutating-tools.ts` — add new mutating tools there (the derived `MUTATING_TOOL_NAMES` set also feeds the `batch_read` blocklist; a drift-guard test enforces coverage). Guard rejections log `outcome=denied`.
- **Connection**: Per-session `ADTClient` pool (`src/adt-client.ts`). `getClient()` returns an implicit single "default" session auto-registered from the `SAP_*` env vars — stdio/local behaviour is unchanged. Multi-user / HTTP callers use `registerSession(key, creds)` + `getClientFor(key)` so each user gets an isolated ADT login (own locks, own SAP audit identity). The network transport (proxy agent + connectivity JWT) is built once via `getSharedAgent()` and shared across all sessions; pool lifecycle helpers: `dropClientSession`, `evictIdleSessions`, `hasSession`, `sessionCount`.
- **Transport**: stdio-based MCP protocol with `@modelcontextprotocol/sdk` (local). An HTTP/Streamable transport for hosted multi-user deployment (e.g. Cloud Foundry) is a planned add-on — see `Updates.md` 2026-06-22.

### Concurrency & Session Management
- **Write Lock**: Serial execution of write operations (`withWriteLock()`) to prevent concurrent ADT lock conflicts. Currently a module-global lock — correct for the single default session; per-client (`WeakMap`-keyed) locking is the planned next step for true multi-user isolation.
- **Stateful Sessions**: Write workflows use stateful mode for complex lock → write → activate sequences (`withStatefulSession(client, fn)` — already client-scoped)
- **Lock Recovery**: Automatic retry logic for stale locks (drops session, full logout/login if needed)

### Tool Architecture
- **Schema Validation**: Zod for all tool parameters (30+ schemas in `src/schemas.ts`)
- **Tool Groups**: SEARCH, READ, WRITE, CREATE, DELETE, TEST, QUALITY, DIAGNOSTICS, TRANSPORT, ABAPGIT, QUERY, DOCUMENTATION, WEBSEARCH, BATCH, ANALYSIS (call graph, dead-code), INTENT (consolidated verbs)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DimiDR/ABAP-MCP](https://github.com/DimiDR/ABAP-MCP) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
