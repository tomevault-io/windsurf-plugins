---
trigger: always_on
description: Guide for AI agents working on this codebase. Read this before writing any code.
---

# AGENTS.md — CoreScope

Guide for AI agents working on this codebase. Read this before writing any code.

## Architecture

Go backend + static frontend. No build step. No framework. No bundler.

**⚠️ The Node.js server (server.js) is DEPRECATED and has been removed. All backend code is in Go.**
**⚠️ DO NOT create or modify any Node.js server files. All backend changes go in `cmd/server/` or `cmd/ingestor/`.**

```
cmd/server/        — Go API server (REST + WebSocket broadcast + static file serving)
  main.go          — Entry point, flags, SPA handler
  routes.go        — All /api/* endpoints
  store.go         — In-memory packet store + analytics + SQLite queries
  config.go        — Configuration loading
  decoder.go       — MeshCore packet decoder
cmd/ingestor/      — Go MQTT ingestor (separate binary, writes to shared SQLite DB)
public/            — Frontend (vanilla JS, one file per page) — ACTIVE, NOT DEPRECATED
  app.js           — SPA router, shared globals, theme loading
  roles.js         — ROLE_COLORS, TYPE_COLORS, health thresholds, shared helpers
  nodes.js         — Nodes list + side pane + full detail page
  map.js           — Leaflet map with markers, legend, filters
  packets.js       — Packets table + detail pane + hex breakdown
  packet-filter.js — Wireshark-style filter engine (standalone, testable)
  customize.js     — Theme customizer panel (self-contained IIFE)
  analytics.js     — Analytics tabs (RF, topology, hash issues, etc.)
  channels.js      — Channel message viewer
  live.js          — Live packet feed + VCR mode
  home.js          — Home/onboarding page
  hop-resolver.js  — Client-side hop prefix → node name resolution
  style.css        — Main styles, CSS variables for theming
  live.css         — Live page styles
  home.css         — Home page styles
  index.html       — SPA shell, script/style tags with __BUST__ placeholder (auto-replaced at server startup)
test-fixtures/     — Real data SQLite fixture from staging (used for E2E tests)
scripts/           — Tooling (coverage collector, fixture capture, frontend instrumentation)
```

### Data Flow
1. MQTT brokers → Go ingestor (`cmd/ingestor/`) ingests packets → decodes → writes to SQLite
2. Go server (`cmd/server/`) polls SQLite for new packets, broadcasts via WebSocket
3. Frontend fetches via REST API (`/api/*`), filters/sorts client-side

### What's Deprecated (DO NOT TOUCH)
The following were part of the old Node.js backend and have been removed:
- `server.js`, `db.js`, `decoder.js`, `server-helpers.js`, `packet-store.js`, `iata-coords.js`
- All `test-server-*.js`, `test-decoder*.js`, `test-db*.js`, `test-regional*.js` files
- If you see references to these in comments or docs, they're stale — ignore them

## Rules — Read These First

### 0. Performance is a feature — not an afterthought
Every change must consider performance impact BEFORE implementation. This codebase handles 30K+ packets, 2K+ nodes, and real-time WebSocket updates. A single O(n²) loop or per-item API call can freeze the UI or stall the server.

**Before writing code, ask:**
- What's the worst-case data size this code will process?
- Am I adding work inside a hot loop (render, ingest, WS broadcast)?
- Am I fetching from the server what I could compute client-side?
- Am I recomputing something that could be cached/incremental?
- Does my change invalidate caches more broadly than necessary?

**Hard rules:**
- **No per-item API calls.** Fetch bulk, filter client-side.
- **No O(n²) in hot paths.** Use Maps/Sets for lookups, not nested array scans.
- **No full DOM rebuilds.** Diff or virtualize — never innerHTML entire tables.
- **No unbounded data structures.** Every map/slice/array must have eviction or size limits.
- **No expensive work under locks.** Copy data under lock, process outside.
- **Cache expensive computations.** Invalidate surgically, not globally.
- **Debounce/coalesce rapid events.** WebSocket messages, scroll, resize — never fire raw.

**If your change touches a hot path (packet rendering, ingest, analytics), include a perf justification in the PR description:** what the complexity is, what the expected scale is, and why it won't degrade.

**Perf claims require proof.** "This is faster" without data is not acceptable. Every PR claiming to fix or improve performance MUST include one of:
- A benchmark test (before/after timings with realistic data sizes)
- Profile output or timing measurements (e.g. "renderTableRows: 450ms → 12ms on 30K packets")
- A test assertion that enforces the perf characteristic (e.g. "filters 30K packets in <50ms")
No proof = no merge.

### 1. No commit without tests
Every change that touches logic MUST have tests. For Go backend: `cd cmd/server && go test ./...` and `cd cmd/ingestor && go test ./...`. For frontend: `node test-packet-filter.js && node test-aging.js && node test-frontend-helpers.js`. If you add new logic, add tests. No exceptions.

### 2. No commit without browser validation
After pushing, verify the change works in an actual browser. Use `browser profile=openclaw` against the running instance. Take a screenshot if the change is visual. If you can't validate it, say so — don't claim it works.

### 3. Cache busters are automatic — do NOT manually edit them

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Kpa-clawbot/CoreScope](https://github.com/Kpa-clawbot/CoreScope) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
