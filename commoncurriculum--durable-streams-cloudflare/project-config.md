---
trigger: always_on
description: Durable Streams on Cloudflare — a port of the [Durable Streams](https://github.com/electric-sql/durable-streams) protocol to Cloudflare Workers + Durable Objects, plus a pub/sub subscription layer on top.
---

# Agent Development Guidelines

## What This Repo Is

Durable Streams on Cloudflare — a port of the [Durable Streams](https://github.com/electric-sql/durable-streams) protocol to Cloudflare Workers + Durable Objects, plus a pub/sub subscription layer on top.

Single unified Worker with all functionality.

## Packages

| Package           | What                                                                                                                                                             |
| ----------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `packages/server` | Durable Streams protocol + pub/sub fan-out. One DO per stream, SQLite hot log, R2 cold segments, subscriptions, sessions, TTL cleanup, Analytics Engine metrics. |
| `packages/docs`   | Slidev presentations.                                                                                                                                            |
| `packages/cli`    | Setup wizard and project management.                                                                                                                             |

Each package has its own `README.md`, `package.json`, `wrangler.toml`, and vitest configs. **Read those directly** — don't rely on this file for their contents.

## Where to Find Things

- **Architecture**: `packages/server/call-graph.md` — request flow diagrams, DO communication patterns
- **API endpoints**: `packages/server/README.md` documents all routes
- **Code organization**: `packages/server/src/http/v1/streams/` — all stream operations
- **Auth patterns**: each package README has auth examples
- **Env vars and wrangler bindings**: each package's `wrangler.toml` and README
- **CI**: `.github/workflows/ci.yml`

## Tech Stack

- **Runtime**: Cloudflare Workers + Durable Objects (SQLite) + R2 + Analytics Engine
- **HTTP**: Hono v4 + ArkType v2 + `@hono/arktype-validator`
- **Build**: TypeScript strict via `tsc` (shared `tsconfig.build.json` at root)
- **Test**: Vitest. Server has 3 vitest configs (unit, integration, conformance). Integration tests use wrangler `unstable_dev`.
- **Lint/Format**: oxlint + oxfmt (NOT ESLint/Prettier)
- **Package Manager**: pnpm (see `packageManager` in root `package.json` for exact version)

## Critical Design Constraints

- **Edge request collapsing is a CORE design goal.** The entire point of the edge cache layer (`caches.default` in the edge worker) is to collapse concurrent reads at the same stream position into a single DO round-trip. Without this, the system cannot scale fan-out reads — 1M followers of a stream means 1M hits to the Durable Object, which is unacceptable. Any change to the edge cache must preserve (or improve) collapsing for live tail long-poll reads.

## Key Conventions

- Server worker pattern: `createStreamWorker()` factory in `src/http/router.ts` + DO classes in `src/http/v1/streams/index.ts` (StreamDO), `src/subscriptions/do.ts` (SubscriptionDO), `src/estuary/do.ts` (EstuaryDO). Entry point is `src/http/worker.ts`.
- Edge worker (`router.ts`) handles: auth, CORS, edge caching, routing to correct DO
- DOs handle: all state mutations, SQLite operations, broadcasts, serialization via `blockConcurrencyWhile`

## Testing

- **Server tests**: `pnpm -C packages/server test` runs integration tests. Use `pnpm test:unit` for pure function tests, `pnpm conformance` for protocol conformance.
- Integration tests start real wrangler workers via `global-setup.ts` files in test directories.
- **Cloudflare Vitest integration**: Use `@cloudflare/vitest-pool-workers` for tests that need Cloudflare runtime APIs (DurableObject, WorkerEntrypoint, bindings, etc.) without mocking. Docs: https://developers.cloudflare.com/workers/testing/vitest-integration/test-apis/
- **Miniflare**: Local simulator for Workers runtime, used under the hood by `wrangler dev` and `@cloudflare/vitest-pool-workers`. Docs: https://developers.cloudflare.com/workers/testing/miniflare/

### Test Patterns

**Unit tests** should use `worker.app.request()` per [Hono testing docs](https://hono.dev/docs/guides/testing):

```typescript
const response = await worker.app.request(
  "/v1/stream/test",
  { method: "PUT", headers: { "Content-Type": "text/plain" } },
  env,
);
```

**Note**: Some existing unit tests use the older `worker.fetch!()` pattern. See `REFACTOR_TESTS_PROMPT.md` for refactoring these to the cleaner `app.request()` pattern.

**Integration tests** use `fetch` with helper utilities from `test/implementation/helpers.ts`:

```typescript
const client = createClient();
const streamId = uniqueStreamId("test");
await client.createStream(streamId, "", "text/plain");
const response = await fetch(client.streamUrl(streamId));
```

### Coverage

**Current Overall Coverage: 80.14% lines** (1849/2307 lines covered)

For complete coverage documentation, see **`packages/server/COVERAGE.md`**.

**Quick reference:**

```bash
# 1. ALWAYS run fresh coverage first (60-90 seconds)
pnpm -C packages/server cov

# 2. Show uncovered lines

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [commoncurriculum/durable-streams-cloudflare](https://github.com/commoncurriculum/durable-streams-cloudflare) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
