---
trigger: always_on
description: <!-- GSD:project-start source:PROJECT.md -->
---

<!-- GSD:project-start source:PROJECT.md -->
## Project

**TradeBlocks**

A Next.js 15 application for analyzing options trading performance. Processes CSV exports from Option Omega backtests and live trade logs, calculates comprehensive portfolio statistics, and provides 50+ MCP tools for AI-assisted analysis including strategy profiling and structure-aware analytics. Uses IndexedDB for client-side storage and DuckDB for server-side analytics.

**Core Value:** Accurate, trustworthy portfolio analytics that help traders understand their strategies and make better decisions.

### Constraints

- **Backwards compatibility**: Existing tools must not change behavior
- **Optional adoption**: Massive.com integration is opt-in — users without API key use CSV import as before
- **MCP server**: New tools follow existing registration patterns (Zod schemas, sync middleware, createToolOutput)
- **Storage**: All data in DuckDB for consistency across the analytics layer
- **No new dependencies for core**: Massive API calls use native fetch — no SDK required
<!-- GSD:project-end -->

<!-- GSD:stack-start source:research/STACK.md -->
## Technology Stack

## Context
- Node.js 22.22.0 (confirmed via runtime check)
- TypeScript 5.8.0 with ESM modules
- Jest 30.2.0 with ts-jest 29.4.6 and ESM preset
- Zod 4.3.6 for schema validation
- @duckdb/node-api 1.4.4 for storage
- Project constraint: "No new dependencies for core: Massive API calls use native fetch"
## Recommended Stack
### Core Technologies
| Technology | Version | Purpose | Why Recommended |
|------------|---------|---------|-----------------|
| `fetch` (native) | Node.js built-in (22.x) | HTTP client for Massive.com REST API | Stable and unflagged since Node.js 18. Node.js 22 ships Undici-backed fetch with full AbortController, timeout signal, and streaming support. Zero additional dependency. Project constraint mandates it. Confidence: HIGH (confirmed via `node -e "console.log(typeof fetch)"` → `function`). |
| `AbortSignal.timeout()` | Node.js built-in (22.x) | Per-request timeout on fetch calls | Ships with Node.js 22. `AbortSignal.timeout(ms)` creates a self-managing timeout signal — no manual `setTimeout`/`clearTimeout` needed. Pass directly to `fetch(url, { signal: AbortSignal.timeout(30_000) })`. Confidence: HIGH. |
| Zod 4.3.6 | Already installed | Validate Massive API response shapes | Already in `package.json`. Define a `MassiveBarSchema` and `MassiveResponseSchema`, parse every API response before mapping to DuckDB rows. Fails loudly on schema drift rather than silently inserting garbage. Confidence: HIGH. |
| `process.env.MASSIVE_API_KEY` | N/A | API key injection | Established pattern in codebase — `TRADEBLOCKS_DATA_DIR`, `MARKET_DB_PATH`, `DUCKDB_THREADS` all follow the same pattern. Read at call site, not stored in module scope, so tests can override with `process.env.MASSIVE_API_KEY = 'test-key'` before each test. Confidence: HIGH. |
### Supporting Libraries
| Library | Version | Purpose | When to Use |
|---------|---------|---------|-------------|
| `p-limit` | 6.x | Concurrency control for paginated requests | When fetching multiple date-range chunks in parallel. Limits concurrent requests to 2-3 to avoid hitting Massive rate limits. Install only if parallel chunk fetching is implemented. If fetching sequentially (simpler), this is not needed — defer to the phase where parallelism is actually warranted. |
| `p-retry` | 6.x | Exponential backoff retry for transient HTTP errors | When Massive returns 429 or 5xx. Wraps individual `fetch` calls with configurable retry count + backoff. Install only if retry logic is needed — it can also be hand-rolled (~20 lines) with `AbortSignal.timeout` since the use case is simple. |
### Development Tools
| Tool | Purpose | Notes |
|------|---------|-------|
| Jest 30 `jest.spyOn(globalThis, 'fetch')` | Mock native fetch in unit tests | Preferred pattern over `jest.mock`. Spy directly on `globalThis.fetch`, restore with `jest.restoreAllMocks()` in `afterEach`. Works cleanly with ESM and ts-jest 29. See Testing section below. |
| `Response` / `Request` (built-in) | Construct mock fetch responses | Use `new Response(JSON.stringify(data), { status: 200, headers: { 'Content-Type': 'application/json' } })` to build mock return values — no library needed. |
## Architecture of the New Module
## Installation
# No new core dependencies required — native fetch covers the HTTP client.
# Optional: only if sequential fetch + simple retry proves insufficient
# Dev: no changes needed (Jest 30 + ts-jest already installed)
## Detailed Recommendations by Question
### HTTP Client: Native Fetch
- Adds ~15KB dependency
- CommonJS-first; ESM support requires `import axios from 'axios'` with interop shims
- The project is `"type": "module"` — axios ESM interop has historically caused bundler friction
- ESM-only (good), but still an external dependency
- Its main value prop (retries, pagination helpers) can be accomplished with ~30 lines of hand-rolled code for a structured import pipeline
- Over-engineered for one-off REST calls against a single API
- Node.js 22 fetch is Undici-backed — production quality, not experimental
- `AbortSignal.timeout(30_000)` handles timeouts cleanly

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [davidromeo/tradeblocks](https://github.com/davidromeo/tradeblocks) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
