---
trigger: always_on
description: > **Role:** Document module, mountable into hosts (first consumer: `hiai-amigo`); **design-token source** for the ecosystem. Standalone open-source AI-native knowledge base (Markdown-first, auto-embeddings, self-hostable).
---

# hiai-docs — AGENTS.md

> **Role:** Document module, mountable into hosts (first consumer: `hiai-amigo`); **design-token source** for the ecosystem. Standalone open-source AI-native knowledge base (Markdown-first, auto-embeddings, self-hostable).
> **Status:** ready
> Project documentation lives in README.md, docs/, and AGENTS.md.

## Cheat-sheet — Conventions

- **Runtime:** Bun 1.3.14+ (no Node, no npm, no yarn)
- **Backend:** Elysia 1.4.28+ (ESM-only, TypeScript strict)
- **Frontend:** SvelteKit 2.60+ + Svelte 5.55+ (`runes: true`)
- **UI:** `@hiai/ui` + shadcn-svelte 1.2.7+ (new-york style) + Tailwind CSS v4
- **Editor:** svelte-tiptap + TipTap v3 (WYSIWYG + raw MD toggle)
- **ORM:** Drizzle ORM 0.45.2+
- **Auth:** Better Auth
- **Validation:** Zod (every route validated)
- **DB:** PostgreSQL 18.4 + pgvector (user-scoped via `owner_id`, `tenant_id` reserved)
- **Vector index (optional):** pgvectorscale StreamingDiskANN with SbqCompression, loaded in the unified PostgreSQL image (see `postgres/Dockerfile`)
- **Cache:** Redis 8.6+
- **Storage:** SeaweedFS (S3-compatible)
- **Embeddings:** external embedding API (configurable) + optional self-hosted Ollama; every provider result must be a finite, non-zero 1024-dimensional vector
- **Search:** exact/title, multilingual lexical, fuzzy, vector, adaptive expansion, and GraphRAG channels fused with reciprocal rank fusion (RRF)
- **GraphRAG:** automatic LLM entity extraction + AGE graph expansion in normal search; the operator flag remains a kill switch for degraded deployments
- **Re-embed invariant:** metadata mutations (tag / folder / category rename and delete) MUST trigger re-embed via `backend/src/lib/reembed.ts`.
- **Logging:** Pino
- **Lint:** Biome 2.5+ (`bun run lint`)
- **Tests:** Vitest (`bun test --path-ignore-patterns="*node_modules*"`)
- **Structure:** `backend/src/` (`api/`, `embedding/`, `lib/`) + `frontend/` (SvelteKit) + `packages/db/` (Drizzle)
- **Module boundaries:** `api/` MUST NOT export internal functions · `embedding/` MUST NOT import from `api/` · `lib/` MUST NOT import from `api/` or `embedding/`
- **Env access:** ONLY via `src/lib/config.ts` (Zod); every `CORS_ORIGINS`, `EMBEDDING_*`, `GRAPH_*`, `SEARCH_*`, `HYBRID_*`, `CHUNK_*`, `*_REEMBED_BATCH_SIZE` through `.env`
- **Token import:** `@hiai/ui/styles/tokens.css` (hiai-docs is the token source for the ecosystem)
- **Ports:** API `50700` · frontend dev `50701` · Postgres `5437` · Redis `6384` · SeaweedFS `50702/50703` · Caddy `80/443`
- **No Playwright** — use `agent-browser` for E2E
- **English only** in code, comments, docs, README, AGENTS.md (zero Cyrillic)

## Project Documents

### Core

- `README.md` — project overview, quick start, configuration
- `AGENTS.md` — this file: rules + canonical-document pointer + document index
- `CONTRIBUTING.md` — code style, testing, PR workflow
- `CODE_OF_CONDUCT.md` — community standards
- `SECURITY.md` — vulnerability reporting
- `CHANGELOG.md` — release notes and breaking-change narrative
- `LICENSE` — Apache-2.0 license

### Project-specific

- [`docs/README.md`](docs/README.md) — documentation index
- [`docs/USAGE.md`](docs/USAGE.md) — product usage, imports, and shortcuts
- [`docs/API.md`](docs/API.md) — REST API and authentication reference
- [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md) — services, data isolation, search, and embedding pipeline
- [`docs/DEPLOYMENT.md`](docs/DEPLOYMENT.md) — Docker and production operations
- [`docs/EXTENDING.md`](docs/EXTENDING.md) — supported UI extension points
- [`docs/RELEASING.md`](docs/RELEASING.md) — evergreen maintainer release flow
- [`docs/openapi.json`](docs/openapi.json) — machine-readable HTTP contract
- `init.sql` — infrastructure bootstrap

## Runtime Contract

| Property | Value |
|----------|-------|
| **Runtime** | Bun 1.3.14+ |
| **Backend** | Elysia 1.4.28+ (ESM-only) |
| **Frontend** | SvelteKit 2.60+ + Svelte 5.55+ |
| **UI** | shadcn-svelte 1.2.7+ (new-york style) + Tailwind CSS v4 |
| **Editor** | svelte-tiptap + TipTap v3 |
| **ORM** | Drizzle ORM 0.45.2+ |
| **Database** | PostgreSQL 18.4 + pgvector |
| **Cache** | Redis 8.6+ |
| **Auth** | Better Auth |
| **Storage** | SeaweedFS (S3-compatible) |
| **Embeddings** | External embedding API (configurable, optional self-hosted Ollama) |
| **GraphRAG** | Automatic LLM entity extraction + AGE traversal in normal search; graceful degradation when unavailable |
| **Logging** | Pino |
| **Validation** | Zod |
| **API port** | 50700 |
| **Frontend port** | 50701 |
| **Module system** | ESM-only, TypeScript strict |

## Canonical Commands

| Task | Command | Working dir |
|------|---------|-------------|
| **Install** | `bun install` | root |
| **Dev (all)** | `bun run dev` | root |
| **Dev (api)** | `bun run dev` | `backend/` |
| **Dev (web)** | `bun run dev` | `frontend/` |
| **Lint** | `bun run lint` | root |
| **Typecheck** | `bun run typecheck` | root |
| **Test** | `bun test` | `backend/` or `frontend/` |
| **DB push** | `bun run db:push` | `packages/db/` |
| **DB generate** | `bun run db:generate` | `packages/db/` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [HiAi-gg/docsmint](https://github.com/HiAi-gg/docsmint) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->
