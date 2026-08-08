---
trigger: always_on
description: Agent guide for **tracer** — a scientific OTLP trace observatory. A Bun API
---

# AGENTS.md

Agent guide for **tracer** — a scientific OTLP trace observatory. A Bun API
server (the middle layer between Grafana Tempo and clients) serves a React
SPA plus an agent-first REST API. Each node of a distributed system (e.g. a
`commonware` consensus cluster) emits its OWN trace; tracer correlates the same
span across those separate traces by name + attribute and renders one
multi-instance comparison (lanes per node).

## Commands

Web + server (`cd web`, bun is the package manager):

- `bun run dev` — Vite dev server (proxies `/api/*` → `localhost:7777`,
  `/tempo/*` → `localhost:3200`)
- `bun run dev:api` — the API server on :7777 (TEMPO_URL defaults to
  `localhost:3200` here; required everywhere else)
- `bun run check` — `tsc --noEmit` (the lint gate; no eslint configured).
  Also the schema drift gate: the `_Check*`/`SchemaDriftChecks` asserts in
  `src/lib/apischema.ts` fail compilation when a JSON Schema and its model
  type diverge.
- `bun test src server` — unit tests (`bun:test`)
- `bun run build` — `tsc --noEmit && vite build`

CI (`.github/workflows/ci.yml`) runs check → test → build on push/PR. Run all
three locally before handing work back.

Docker / demo (`cd docker`, uses `just`):

- `just build` — build the prod image (`tracer-web:local`): one Bun process
  serving the SPA + `/api/v1` + the GET-only `/tempo` passthrough
- `just app <tempo-url>` — run the standalone viewer/API against a Tempo endpoint
- `just demo` / `just demo-down` / `just demo-logs` / `just clean` — the local
  demo stack (Tempo + 4 `consensus-sim` nodes + viewer) on http://localhost:8080

Loadgen: `cd docker/demo/loadgen && cargo check` (single binary crate
`consensus-sim`; must compile with zero errors/warnings).

Version control is **jujutsu (`jj`)**, not plain git. Make each logical change
its own revision (`jj new -m "…"`); the working copy is always a commit.

## Repository layout

```
web/                  TypeScript + React 19 + Vite SPA, plus the Bun API server
  src/lib/model.ts    AUTHORITATIVE shared types — never redefine these shapes
  src/lib/wire.ts     JSON-safe wire encoding of TraceModel (+ aggregate flatten)
  src/lib/apischema.ts JSON Schemas for the API — single source for openapi.json,
                       compile-time-checked against model.ts/wire.ts types
  src/lib/format.ts   duration/time formatting + parsing helpers
  src/lib/trace.ts    OTLP JSON → TraceModel parser + aggregate flame tree
  src/lib/traceql.ts  FilterState → TraceQL compiler
  src/lib/range.ts    time-range presets + resolution
  src/api/tempo.ts    TempoClient — the server-side Tempo engine (windowed
                       search, v2→v1 fallbacks); no longer imported by the SPA
  src/api/client.ts   ApiClient (implements ITempoClient) — what the SPA uses
  src/components/      SearchPanel, Combobox, TraceList, FlameGraph, SpanStats,
                       HeatMap, SpanDetails, EventsView, RangePicker, Select,
                       ExportModal, Calendar (each with a co-located .css)
  src/App.tsx          shell: routing, state (written last)
  server/              the API server (Bun.serve; zero runtime deps)
    index.ts           entry: dispatch, static SPA, /tempo passthrough, errors
    routes.ts          THE route table — the only way to expose a route
    surface.ts         composed table (discovery routes + data routes)
    params.ts          GET dialect + POST body → FilterState/TimeRange
    search.ts traces.ts tags.ts misc.ts   handlers
    discovery.ts openapi.ts docs.ts       /api/v1 index, openapi.json, llms.txt
    registry.ts        published schema set; problem.ts — RFC 9457 errors
skills/tracer-api/    Claude Code skill for querying a deployed tracer API
docker/               prod image: Dockerfile.web (bun runtime), bake, justfile
docker/demo/          demo stack: compose, tempo.yaml, loadgen/ (consensus-sim)
.github/workflows/    ci.yml (build) + docker.yml (multi-arch release on v* tags)
```

## Hard rules

- **Types live in `web/src/lib/model.ts`** (wire forms in `lib/wire.ts`).
  Import them; never redefine a shape. Components implement exactly the
  `*Props` in model.ts.
- **API responses are schema-bound.** Every route body has a JSON Schema in
  `lib/apischema.ts`; schemas and types are locked together by compile-time
  asserts. New/changed routes go through `server/routes.ts` (the table feeds
  dispatch, the discovery index, AND openapi.json) with summary, params, and
  a runnable example. Non-2xx responses are `application/problem+json`.
- **Colors come from `web/src/styles/tokens.css` only** — never hardcode. Canvas
  code reads resolved values via `getComputedStyle` (re-read on theme change).
  Instance colors are generated per service name (`colorIndexForService` →
  `instanceColorVar` → an `hsl()` hue at the theme's `--instance-sat`/
  `--instance-lum`); levels use `--level-{name}`.
- **No new runtime deps** beyond package.json / Cargo.toml without strong reason.
  No UI component libraries; no virtualization deps. The server has ZERO runtime
  deps (`json-schema-to-ts` is type-only).
- Co-locate `ComponentName.css`; prefix classes by component (`.fg-`, `.sp-`,
  `.tl-`, `.hm-`, …). Reuse `global.css` classes: `.btn(.btn-primary/-ghost/-sm)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [clabby/tracer](https://github.com/clabby/tracer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-07 -->
