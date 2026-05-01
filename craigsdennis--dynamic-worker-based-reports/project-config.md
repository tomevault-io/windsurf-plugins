---
trigger: always_on
description: Guidance for AI coding agents working on this project.
---

# AGENTS.md — Dynamic Workers Report Builder

Guidance for AI coding agents working on this project.

---

## What This Project Does

A demo app for the "Run Code That Writes Itself" YouTube video. A user types a natural-language prompt, Workers AI generates a JavaScript Cloudflare Worker, that Worker is compiled and run as a Dynamic Worker with a controlled data binding, and an HTML report with an interactive chart appears.

The key Cloudflare primitives in play:

| Primitive | Binding | Purpose |
|---|---|---|
| Workers AI | `env.AI` | Generates the report Worker code from a prompt |
| Worker Loader (Dynamic Workers) | `env.LOADER` | Compiles and runs AI-generated code at runtime |
| Workers RPC / WorkerEntrypoint | `DataService` via `ctx.exports` | Exposes the survey dataset to the dynamic worker |
| KV | `env.REPORTS` | Stores saved report metadata + source code |
| Durable Objects | `env.LOG_SESSION` | Log capture pipeline (implemented, not yet wired to LOADER) |

---

## Architecture

```
Browser (React + Vite)
    │
    └── POST /api/run  (SSE stream)
            │
            ▼
    Loader Worker  (src/index.ts)
            │
            ├── env.AI.run()               → nemotron generates JS worker code
            ├── createWorker({ files })    → @cloudflare/worker-bundler bundles it
            ├── env.LOADER.get(id, cb)     → Dynamic Worker loaded and cached
            └── ctx.exports.DataService() → RPC binding injected into dynamic worker
                        │
                        ▼
            Dynamic Worker  (AI-generated)
                ├── env.DATA.getAIData()  → fetches survey data via RPC
                └── returns HTML page with Chart.js CDN charts (client-side rendering)
```

Saved reports are stored in KV under two keys:
- `id:<workerId>` — looked up by list/kill endpoints
- `slug:<slug>` — looked up by `/r/:slug` route

On a cache miss (isolate evicted), the loader re-bundles from the `fullCode` stored in KV.

---

## File Structure

```
src/
  index.ts              # Loader Worker — all API routes + Dynamic Worker orchestration
  data-service.ts       # DataService WorkerEntrypoint + 82-record SO survey dataset
  log-session.ts        # LogSession DO + DynamicWorkerTail (log capture pipeline)
  main.tsx              # React entry point
  ui/
    App.tsx             # Root component — state, API calls, layout
    PromptPanel.tsx     # Left panel: prompt input, binding toggle, Run/Save
    OutputPanel.tsx     # Right panel: iframe report, generated code tab, logs, timing
    ReportList.tsx      # Bottom panel: saved report cards with slug URLs
wrangler.jsonc          # Worker config
vite.config.ts          # Vite config (proxies /api/* and /r/* to Worker in dev)
env.d.ts                # Type references for wrangler-generated types
```

---

## Running Locally

Two terminals are required — Dynamic Workers and Workers AI don't work with the local Wrangler simulator, so the worker runs against real Cloudflare infrastructure.

```bash
# Terminal 1 — Vite dev server (UI, proxies API calls to Worker)
npm run dev:ui

# Terminal 2 — Wrangler dev against Cloudflare (remote mode)
npm run dev:worker
```

Open `http://localhost:5173`.

---

## Deploying

```bash
npm run deploy          # vite build + wrangler deploy
```

The `REPORTS` KV namespace is auto-provisioned on first deploy if it doesn't exist (no `id` in `wrangler.jsonc`). After first deploy, wrangler writes the real ID back into the config.

After any change to `wrangler.jsonc` bindings, regenerate types:

```bash
npm run types           # wrangler types
```

---

## Key Implementation Details

### Dynamic Worker Lifecycle

1. **Generate** — `env.AI.run(nemotron, { messages, stream: true })` streams the worker code. If streaming returns empty (nemotron quirk), a non-streaming fallback call is made.
2. **Strip** — `buildDynamicWorker()` removes any `import` statements the model may have added. The generated code must be self-contained.
3. **Bundle** — `createWorker({ files: { "src/index.js": code } })` from `@cloudflare/worker-bundler` compiles and bundles the code.
4. **Cache** — `env.LOADER.get(workerId, callback)` returns a warm isolate if the same worker ID was seen recently. Worker IDs are content-hashes of the generated code + binding flag, so the same prompt + same binding always hits the warm cache.
5. **Run** — `worker.getEntrypoint().fetch(request)` executes the dynamic worker.

### Why Chart.js (not vega-lite)

Vega/vega-lite were attempted but are incompatible with the Dynamic Worker runtime. The `@cloudflare/worker-bundler` output is a CJS/ESM hybrid; vega's transitive dependencies import Node built-ins (`stream`, `node:http`) as bare ESM specifiers. With `nodejs_compat` enabled, the esbuild `__require` shim bypasses its bundled registry and calls the real `require()`, which can't find `vega-lite`. Without it, the Node built-in imports fail. No combination of compatibility flags resolves both.

Chart.js avoids this entirely: the dynamic worker returns plain HTML with a `<script src="https://cdn.jsdelivr.net/npm/chart.js@4/...">` tag, and all charting happens client-side in the browser. The iframe has `sandbox="allow-scripts"` to permit this.

### The DATA Binding


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [craigsdennis/dynamic-worker-based-reports](https://github.com/craigsdennis/dynamic-worker-based-reports) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-26 -->
