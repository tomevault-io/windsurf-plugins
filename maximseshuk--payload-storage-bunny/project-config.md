---
trigger: always_on
description: Payload CMS 3.x storage adapter for Bunny.net. Wraps `@payloadcms/plugin-cloud-storage` and adds:
---

# Bunny.net Storage for Payload

Payload CMS 3.x storage adapter for Bunny.net. Wraps `@payloadcms/plugin-cloud-storage` and adds:

- **Bunny Storage** — files, images, documents (HTTP API or S3-compatible).
- **Bunny Stream** — video with HLS/MP4, thumbnails, TUS resumable uploads.
- **Client-direct uploads** — browser → Bunny (presigned S3 or an Edge Script).
- **Signed URLs** — time-limited links with country and per-client IPv4 locking.
- **CDN cache purging** — auto-invalidate on upload/delete.
- **Per-collection overrides** — every setting, plus a collection's own zone/library.
- **CLI** — `init` setup wizard and `bunny:deploy-edge-script`.
- **Subpath exports** — `./client` (admin UI), `./media-preview` (Stream adapter), `./migrations` (v2→v3 data migration).

## Environment

- Package manager is **pnpm**. Node.js 22+, Payload CMS 3.83+.
- Install with `pnpm install`.
- Runtime commands that touch Bunny read secrets from `.env` (loaded via `dotenv`); never hardcode keys.
- Agent skills are **not vendored** — only `skills-lock.json` is committed (it pins each skill by content hash). Restore them with `npx skills experimental_install`; nothing in the build, tests or CI depends on them.

## Commands

```bash
pnpm typecheck        # tsc --noEmit — ALWAYS run before committing
pnpm lint             # oxlint — add `-f agent` for compact AI-readable output
pnpm lint:fix         # oxlint --fix
pnpm format           # oxfmt (write)
pnpm format:check     # oxfmt --check

pnpm build            # tsdown (bundles dist/)
pnpm clean            # remove dist/ + tsbuildinfo

pnpm test:unit        # vitest, no env — the default fast gate
pnpm test             # vitest with .env
pnpm test:coverage    # vitest + coverage
pnpm test:e2e         # live e2e against real Bunny resources (needs .env)

pnpm dev              # dev/test Payload app (tests/dev.ts)
pnpm docs:dev         # Mintlify docs site (docs/)
pnpm docs:openapi     # regenerate docs/api-reference/openapi.json from src/server/payload/openapi.ts
pnpm docs:validate    # mint validate (MDX + build check)
```

Run `pnpm typecheck && pnpm lint && pnpm format` before every commit. Not just typecheck.

When you run the linter yourself, use **`pnpm lint -f agent`** — oxlint's compact, AI-readable format (`file:line: level rule msg`). CI needs no flag: oxlint auto-detects GitHub Actions and emits annotation (`github`) format.

## Repository Structure

Three entrypoints — `index.ts` (server), `client/index.ts` (admin UI), `cli/index.ts` (bin) — and four buckets. Dependency direction inside `server/`: payload → bunny → http → shared.

```
src/
├── index.ts        # Server entry — plugin that extends Payload config
├── shared/          # isomorphic leaf: types/ (config.ts user-facing JSDoc, configNormalized.ts, core.ts), translations/, constants.ts, mimeTypes.ts, http.ts, urlTransform.ts, zoneSecret.ts
├── client/          # index.ts = 'use client' entry (./client subpath); TusUpload/* (upload button), ClientUploadHandler
├── edge/            # uploader.edge.js — Bunny Edge Script for client-direct uploads
├── cli/             # index.ts = bin entry (`init` wizard, args via `cac`); commands/ (init/ wizard, deployEdgeScript.ts), lib/ (shared Logger, bunnyFetch, envFile — reuse, don't duplicate)
└── server/
    ├── http/        # lowest server leaf — the shared ky client for every outbound request
    ├── bunny/       # the only Bunny HTTP API layer — client.ts, storage.ts, s3.ts, stream.ts, cdn.ts
    ├── payload/     # config/ (normalizer, context, access, defaults, validator), fields/, storage/ (+ clientUploads/), stream/, migrations/, openapi.ts, tokenAuth.ts, mediaPreview.ts
    ├── telemetry/   # anonymous opt-out usage telemetry — fired from index.ts onInit; depends only on @/shared + node builtins, imported only by the plugin entry
    ├── urls.ts      # URL builders
    └── files.ts     # filesystem helpers
```

The `telemetry` plugin option (`boolean | { endpoint?: string }`) reads from `config._original.telemetry`; it needs no normalizer entry. Feature flags are derived in `server/telemetry/features.ts` from the resolved `NormalizedBunnyStorageConfig` (booleans only — never zone/library/collection names or other values).

`server/payload/openapi.ts` is the single source for the OpenAPI doc; `pnpm docs:openapi` writes docs/api-reference/openapi.json via scripts/build-openapi.ts. `migrations/` backs the ./migrations subpath, `mediaPreview.ts` the ./media-preview Stream adapter.

## Architecture

Data flow:

```
User Config → Normalizer → Collection Context → Adapter → Bunny API
```

1. **User config** (`BunnyStorageConfig`) — what the user writes in `payload.config.ts`.
2. **Normalizer** (`server/payload/config/normalizer.ts`) — fills defaults, validates, and merges global + per-collection overrides (the `resolveCollection*Config` family). **Apply overrides here.**
3. **Collection context** (`server/payload/config/context.ts`) — wraps the already-resolved per-collection config as the runtime `CollectionContext`. No merging here.
4. **Adapter** (`server/payload/storage/*`, `server/payload/stream/*`) — consumes the context.
5. **Bunny API** — Storage or Stream endpoints.

### Always use collection context


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [maximseshuk/payload-storage-bunny](https://github.com/maximseshuk/payload-storage-bunny) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
