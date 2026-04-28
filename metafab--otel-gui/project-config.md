---
trigger: always_on
description: A lightweight OpenTelemetry trace viewer built with SvelteKit 5. Port 4318 (OTLP/HTTP standard).
---

# otel-gui Agent Instructions

A lightweight OpenTelemetry trace viewer built with SvelteKit 5. Port 4318 (OTLP/HTTP standard).

## Build & Dev

```sh
pnpm run dev        # Start on port 4318
pnpm run check      # Type-check before commits
pnpm run test       # Run unit tests (Vitest)
pnpm run test:watch # Tests in watch mode
```

## Formatting Workflow

Before submitting changes (including agent-generated edits), run:

```sh
pnpm run format:changed
pnpm run lint:fix
```

Then validate with:

```sh
pnpm run format:check
```

**Required**: `pnpm` (not npm/yarn), Node.js 20+, `@sveltejs/adapter-node` (persistent server process state)

## Architecture

- **Routes**: `/v1/traces` + `/v1/logs` (OTLP receivers), `/api/traces` (list + clear/delete-selected), `/api/traces/:id` (detail), `/api/traces/:id/export` (single export), `/api/traces/export` (bulk export), `/api/traces/import/preview` + `/api/traces/import` (import flow), `/api/service-map` (service graph), `/api/config` (runtime config + persistence status)
- **Server-only code**: `$lib/server/` — never bundled for client
- **Utilities**: `$lib/utils/` — shared helpers for OTLP data transformation
- **Types**: `$lib/types.ts` — all interfaces (use `import type`)

## OTLP Data Handling

### Critical Patterns

**Timestamps are nanosecond strings** (not numbers):

```typescript
// Use BigInt for arithmetic, convert only after scaling
const durationNs = BigInt(endNano) - BigInt(startNano)
const durationMs = Number(durationNs / 1_000_000n)
```

See [time.ts](src/lib/utils/time.ts) for all time formatting.

**Flatten KeyValue[] to Record<string, any>**:

```typescript
// OTLP: [{ key: "http.method", value: { stringValue: "GET" } }]
// →: { "http.method": "GET" }
```

Use `flattenAttributes()` from `@otel-gui/core` ([packages/core/src/attributes.ts](packages/core/src/attributes.ts)) — handles all 7 AnyValue variants.

**Service name extraction**: Lives in `ResourceSpans.resource.attributes['service.name']`, not in spans. Must propagate during ingestion.

**Scope attributes**: `InstrumentationScope.attributes` are flattened and stored as `scopeAttributes` on each `StoredSpan` (alongside `scopeName` and `scopeVersion`). The span detail sidebar shows them in a collapsible **Scope** section below Attributes.

**Span merging**: Traces arrive incrementally across multiple POST requests. Store merges spans by `traceId`, handles out-of-order root spans. The merge logic lives in [core.ts](src/lib/server/traceStore/core.ts) and is used by swappable backends.

## Optional Persistence (PGlite)

`traceStore` now supports pluggable backends:

- `memory` (default, OSS built-in)
- `pglite` (optional, typically provided by an external enterprise module)

Environment variables used by the bootstrap layer in `traceStore.ts`:

- `OTEL_GUI_PERSISTENCE_MODE`: `memory` or `pglite` (invalid values warn and fall back to `memory`)
- `OTEL_GUI_PERSISTENCE_PATH`: local PGlite data path (default `.otel-gui/pglite`)
- `OTEL_GUI_PERSISTENCE_FLUSH_MS`: flush debounce in ms (50-60000, default `750`)
- `OTEL_GUI_PERSISTENCE_BACKEND_MODULE`: optional module id/path dynamically imported to register extra backends

The runtime exposes persistence status via `getPersistenceStatus()` and `GET /api/config`:

- `mode`, `enabled`, `backend`, `path`, `flushMs`
- `lastRestoreAt`, `restoredTraceCount`, `pendingFlushCount`
- `unavailableReason` when `pglite` is requested but unavailable or failed

External backend interop detail: before dynamically importing backend modules, OSS hydrates selected license-related env vars from `$env/dynamic/private` into `process.env` (`OTEL_GUI_LICENSE_*`) so external modules reading `process.env` still work.

## Svelte 5 Runes (Planned)

```typescript
// Client stores (.svelte.ts files)
let traces = $state.raw<TraceListItem[]>([]) // .raw = no deep proxying for large arrays
let selectedId = $state<string | null>(null)
let selected = $derived(traces.find((t) => t.id === selectedId))

$effect(() => {
  // Polling, subscriptions
})
```

**Why `$state.raw`**: Trace lists are large arrays replaced wholesale — avoid deep reactive proxy overhead.

## Testing

**Current status**: 284 unit + integration/component tests, all passing. Run with `pnpm run test`.

**Test files**:

| File                                                                              | What's covered                                                                                                                       |
| --------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------ |
| [attributes.test.ts](packages/core/src/attributes.test.ts)                        | All 7 AnyValue variants, null/edge cases (canonical tests in `@otel-gui/core`)                                                       |
| [time.test.ts](src/lib/utils/time.test.ts)                                        | Duration formatting, negative/zero, timestamps, relative time                                                                        |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [metafab/otel-gui](https://github.com/metafab/otel-gui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
