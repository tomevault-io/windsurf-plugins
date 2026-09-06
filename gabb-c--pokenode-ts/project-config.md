---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

`pokenode-ts` — typed PokéAPI client, zero runtime dependencies, built on native `fetch`. Ships dual ESM/CJS from `src/index.ts` to `lib/`. Package manager is **pnpm** (`packageManager` pin); Node >= 22.

## Commands

| Command | What it does |
| --- | --- |
| `pnpm test` | Vitest, watch mode |
| `pnpm test run tests/berry/berry.spec.ts` | Single file, single run (no `--`; pnpm swallows the filter after it) |
| `pnpm test run tests/berry -t "list of berries"` | Single test by name |
| `pnpm test:run` | Vitest, unit project, single run |
| `pnpm typecheck` | `tsc --noEmit` twice: root config (`src`, no Vitest globals) then `tests/tsconfig.json` |
| `pnpm test:live` | Drift check against the real PokéAPI — needs network, not part of CI on PRs |
| `pnpm test:coverage` | Single run + lcov/html coverage |
| `pnpm lint` | Biome check, writes fixes |
| `pnpm lint:ci` | Biome check, no writes — what CI runs |
| `pnpm build` | tsdown build; runs `publint` + `attw` as part of it |
| `pnpm docs:dev` | VitePress docs site locally |

Match CI before opening a PR: `pnpm lint:ci && pnpm typecheck && pnpm test:coverage && pnpm build`.

## Architecture

**Layers.** `src/models/` (pure types mirroring PokéAPI schemas) → `src/constants/` (endpoint paths + name→ID enum maps) → `src/clients/` (one class per PokéAPI section) → `src/index.ts` (single public barrel). `src/utils/` holds public helpers; `src/internal/` holds helpers the barrel deliberately does not export.

**`Transport` (`src/internal/transport.ts`) holds the request pipeline** — requests, caching, coalescing, revalidation and logging. The rules it works to live beside it, one concern per module: `src/internal/url.ts` (trimming, normalization, credential splitting, `toEndpointPath`), `src/internal/retry.ts` (attempt count, `Retry-After`, backoff, abort detection), `src/internal/paginate.ts` (`walk`), `src/internal/pool.ts` (`mapWithConcurrency`), `src/internal/inflight.ts` (`InFlight`, the coalescing of concurrent identical requests). `paginate.ts` takes a `resolve` callback rather than a transport, which is what keeps `transport.ts` ↔ `paginate.ts` from being a runtime cycle; `inflight.ts` takes a `start` callback for the same reason, and knows nothing about HTTP.

**Client classes** (`src/clients/base.ts`) — `ClientFacade` owns the transport and everything that is not an endpoint: `cache`, `with`, `clearCache`, `resolve`, `resolveAll`, the protected `getResource*` helpers and the protected `walk` bridge. `BaseClient extends ClientFacade` and adds only `paginate`; `MainClient extends ClientFacade` and adds only the twelve sections, so `mainClient instanceof BaseClient` stays `false` as 2.0 documented. Every section client is a thin `BaseClient` subclass whose methods just call `getResource(endpoint, id)`, `getResourceByURL(url)`, or `getListResource(endpoint, offset, limit)`. Adding an endpoint means: add the path to `src/constants/endpoints.ts`, the response type under `src/models/`, and a one-line method on the section client. Do not put fetch/cache/logging logic in a section client.

**Keeping `Transport` out of the published types.** `base.ts` owns every public type and `transport.ts` imports them back with `import type` (erased, so the runtime graph stays one-directional). **Public types belong in `base.ts`, never in `src/internal/`** — that is where the barrel points, and `src/internal/` is by definition what the barrel does not export. Not exporting it is not enough on its own: a `protected` member keeps its type in the emitted `.d.ts`. Two things hold the line, and both must stay — `ClientFacade` stores the transport in a `#transport` private field (emitted as `#private;`), and the constructor that accepts one is a second overload tagged `@internal`, stripped by `"stripInternal": true` in `tsconfig.json`. A `build:done` hook in `tsdown.config.ts` enforces this: it strips comments from `lib/index.d.ts` and `lib/index.d.cts` and fails the build if `Transport` survives. TypeScript 7 warns on every build that some options may be unavailable, and `stripInternal` is one of them — the hook is what makes that a build failure rather than a silent leak into a release.

The request pipeline in `Transport.request` is: cache lookup → `InFlight.share` (concurrent identical URLs share one round trip) → `fetch`. Two invariants are load-bearing and documented inline:

- **URL normalization** — `Transport.resource` builds `/berry/1` while `Transport.byURL` receives PokéAPI links ending in `/`. Both are normalized to one cache key or each caches the other's misses. The key is also the credential-stripped URL, so calls to one host under different credentials share a cache entry and a round trip — a client per identity needs a `CacheStore` per identity.
- **URL joining is string concatenation, not `new URL(path, base)`** — the base carries a `/api/v2` path that URL resolution would discard.
- **`toEndpointPath`** re-resolves foreign absolute URLs against the client's own `baseURL` by parsed URL components (a raw-string version-marker search matches hosts like `api.v2.example.com`).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Gabb-c/pokenode-ts](https://github.com/Gabb-c/pokenode-ts) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-06 -->
