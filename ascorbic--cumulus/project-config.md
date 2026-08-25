---
trigger: always_on
description: Guidance for agentic coding tools working in this repository. The README is
---

# Cumulus

Guidance for agentic coding tools working in this repository. The README is
the user-facing documentation; this file is about how the code is built and
what must not break.

Cumulus is an ATProto blob proxy on Cloudflare Workers using **Workers
Cache** (`cache.enabled`, GA July 2026) as its only caching layer. All eight
phases of the original spec are implemented and deployed. The spec and the
per-phase plans with measured results live in `.claude/docs/` — consult
`.claude/docs/SPEC.md` for design rationale and `.claude/docs/plans/` for
what was verified on the platform and what was found along the way.

## Before touching cache-related code

Workers Cache, the Images binding, the `cf` CLI and the experimental
`cloudflare.config.ts` format are all newer than most training data. Read
the current docs rather than writing from memory:

- https://developers.cloudflare.com/workers/cache/ (+ `configuration/`,
  `cache-keys/`, `purge/`, `limitations/`)
- https://developers.cloudflare.com/images/transform-images/bindings/

This project does **not** use `caches.default` / the CacheStorage API
anywhere. `caches.default.put(...)` is the old colo-local API; its presence
is a bug.

## Invariants (violating any of these is a bug, not a style choice)

1. **Every response sets explicit `Cache-Control`** — errors, redirects,
   health checks, admin. Heuristic caching silently caches bare responses.
   Use the `CACHE_CONTROL` constants in `src/response.ts`.
2. **Never return `206`.** The platform slices ranges from stored 200s;
   the 200 must carry `Accept-Ranges: bytes` or slicing does not happen.
3. **Verify before serving.** Blob bytes are buffered and SHA-256-checked
   against the CID before any byte reaches the client or cache.
4. **No query-string routes.** Path-only; param order fragments cache keys.
5. **Tag everything cacheable** — including 403s and 404s — with
   `did:{did}` / `cid:{cid}` (lowercased) plus `v:{versionId}`; scoped
   responses add `rec:{did}/{collection}/{rkey}`; 413/415 add
   `cfg:{hash}`. Untagged deny responses cannot be purged.
6. **Purges fan out per entrypoint** via the `purgeTags`/`purgeEverything`
   RPC methods on `Identity`, `Policy` and `Record`. A purge from `default`
   does not touch another entrypoint's cache. Every `purge()` call counts
   against the (Free-tier) purge rate limit separately.
7. **Content-Type comes from magic-byte sniffing** (`src/sniff.ts`), never
   from the PDS. `image/svg+xml` stays off the allowlist.
8. **No Durable Objects, Workflows or Queues.** Drains are cron + KV cursor;
   overlapping drains are idempotent. Outbound websockets live only for
   the duration of a drain.
9. **Derived responses (`/metadata`, `/img/`) obtain the original via
   `ctx.exports.default.fetch()` loopback**, never by fetching the PDS
   themselves. Presets are the fixed Bluesky four; no free-form parameters.
10. **Scoped-mode admission is a forward `getRecord` membership check**
    through the `Record` entrypoint; never build a reverse index of blob
    references. Open and scoped routes never coexist in one deployment.
11. **Settings are text bindings in `cloudflare.config.ts`** — that file
    is the user-facing configuration. `loadConfig(env)` in `src/config.ts`
    parses them, with `CONFIG_DEFAULTS` as the fallback for environments
    that omit a binding (tests). Adding a setting means touching both plus
    the README table. Nothing reads `process.env` at build time.

## Layout

```
src/index.ts              router for the default entrypoint; exports Identity, Policy, Record; scheduled()
src/entrypoints/          Identity (DID → PDS / labeler endpoint), Policy (verdicts), Record (getRecord + blob refs)
src/blob.ts               PDS fetch, buffering with the size cap, sha256
src/cid.ts                base32 + CIDv1/raw/sha2-256 decode; no multiformats dependency
src/sniff.ts dimensions.ts  magic bytes and header-only dimension parsing for the five formats
src/path.ts scoped.ts img.ts  path parsing and canonicalisation; aliases 301 to one canonical URL
src/response.ts           header contract, tag helpers, purge helpers (ctx.cache may be absent locally)
src/admin.ts              Basic-auth admin routes
src/labels.ts drain.ts    labeler config, queryLabels client, subscribeLabels drain (DAG-CBOR via src/cbor.ts)
src/jetstream.ts socket.ts  Jetstream drain; shared outbound-websocket reader
src/store.ts              the only KV schema: cursors, drain status, record-level deny set
src/config.ts             CONFIG_DEFAULTS and loadConfig
cloudflare.config.ts      settings (text bindings), exports (per-entrypoint cache), Images, KV, cron
test/                     vitest inside workerd; test/integration/ runs on Node against the deployed Worker
```

## Commands

- `pnpm dev` — local dev server (real PLC/PDS, local cache semantics)
- `pnpm test` — workerd suite; `pnpm test:deployed` — HTTP suite against
  production (reads `ADMIN_PASSWORD` from `.env`; it purges, so run it
  sparingly — the purge rate limit bites after a few runs)
- `pnpm check` / `pnpm fix` — oxfmt + oxlint + types
- `pnpm run deploy` — `cf deploy` (plain `pnpm deploy` is a pnpm builtin).
- `pnpm admin:password` — generate the admin secret, set it, save it to `.env`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ascorbic/cumulus](https://github.com/ascorbic/cumulus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-22 -->
