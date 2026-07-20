---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Kherad is an internal, git-backed wiki. Non-technical authors edit pages through a Lexical
block editor that feels like Notion, but every save is a real git commit under the hood, and
every published change goes through a merge-request review/approve step. See `PRD.md` for the
full product/technical spec (roles, data model, workflows) — read it when a task touches
permissions, the git/branching model, or the review workflow, since this file only covers
what's needed to navigate and run the code.

## Commands

This is a pnpm + Turborepo monorepo (`pnpm@10.34.4`, Node >=20). Run from repo root unless noted.

```sh
pnpm dev            # turbo run dev — starts apps/api (tsx watch) and apps/web (next dev) together
pnpm build          # turbo run build
pnpm lint           # turbo run lint
pnpm check-types    # turbo run check-types (tsc --noEmit in every package)
pnpm test           # turbo run test (currently only packages/core has tests)
pnpm format         # prettier --write across the repo
```

Scope any of the above to one workspace with `pnpm --filter <name> <script>` (package names:
`api`, `web`, `@kherad/core`, `@kherad/db`, `@kherad/ui`).

**Tests** live only in `packages/core` (Vitest). Run them from that package:

```sh
cd packages/core
pnpm test                                   # vitest run — all tests
pnpm vitest run src/git/engine.test.ts      # single file
pnpm vitest run -t "some test name"         # single test by name
```

**Database** (`packages/db`), via `drizzle-kit`, run from `packages/db` after `cp .env.example .env`:

```sh
pnpm db:generate   # write a new migration from src/schema.ts
pnpm db:migrate    # apply pending migrations
pnpm db:push       # push schema directly, no migration file (prototyping only)
pnpm db:studio     # Drizzle Studio against DATABASE_URL
pnpm db:seed       # idempotent: one admin user + one public "welcome" bundle
```

**Local infra**: `docker-compose.yml` at the root brings up Postgres, the Python Docling
ingest service, and (if built) the api/web containers. For day-to-day dev, run
`docker-compose up postgres ingest` and run `api`/`web` with `pnpm dev`.

**Document ingest** (`apps/ingest`): FastAPI + Docling conversion microservice on port 4100.
Not part of the pnpm workspace. Locally:

```sh
cd apps/ingest
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
uvicorn main:app --reload --port 4100
```

Set `INGEST_SERVICE_URL=http://localhost:4100` in `apps/api/.env`. OCR uses dedicated admin
settings at `/admin/ocr` (OpenAI-compatible vision). Voice ingest uses `/admin/stt`
(OpenAI-compatible `/audio/transcriptions`). Both are separate from `/admin/ai`.

Each app/package needs its own `.env` (copy from the adjacent `.env.example`): `apps/api`,
`apps/web`, `packages/db`, `packages/core`. Key vars: `DATABASE_URL`, `GIT_REPO_PATH` (api
only — where the bare git repo lives on disk), `JWT_SECRET`, `NEXT_PUBLIC_API_URL` (web →
api base URL), `WEB_ORIGIN` (api CORS allowlist), `INGEST_SERVICE_URL` (api → Docling service).

## Architecture

### Two sources of truth

Page **content** (markdown + binaries) lives only in a single bare git repo on local disk,
read/written exclusively through `packages/core/src/git` (an `isomorphic-git` wrapper) —
never through Postgres. Everything else — users, sessions, permissions, bundles/pages
_metadata_, merge requests, comments, autosave drafts, presence — lives in Postgres via
`packages/db` (Drizzle ORM, schema in `packages/db/src/schema.ts`). A `pages` row's `path`
is the join key into the git tree: `apps/api/src/lib/wiki-paths.ts` maps
`(bundle.slug, page.path)` → `wiki/<bundle-slug>/<page.path>.md`.

### `packages/core` is the shared brain

Both `apps/api` and `apps/web` import `packages/core` directly (workspace package, not an
HTTP call) so there is exactly one implementation of "can this user do X" and "how do we read
a git ref." It has three independent entry points (see its `package.json` `exports`):

- `@kherad/core/git` — the git engine (`createGitEngine`, `packages/core/src/git/engine.ts`).
  All read functions (`getFileAtRef`, `listBranches`, `diffRefs`, …) hit the bare repo directly.
  All write functions (`createUserBranch`, `writeAndCommit`, `squashMerge`) are routed through
  `createWriteLock` (`git/lock.ts`), a two-layer lock: an in-process promise-chain mutex plus an
  OS-level exclusive-file-create lock (`O_EXCL`) on the repo directory, so it's safe even if a
  second process ever touches the same repo. Next.js SSR reads never take this lock; only
  `apps/api` writes go through it.
- `@kherad/core/auth` — `login`/`logout`/`getSession`/`requireRole` plus password hashing
  (`argon2`) and JWT signing (`jose`). **Note:** the current implementation is bearer-JWT
  (`Authorization: Bearer <token>`, stored client-side in `localStorage` under `kherad.token` —
  see `apps/web/src/lib/api-client.ts`), where the JWT's `jti` claim maps to a row in the
  Postgres `sessions` table so logout/expiry still revoke server-side. This differs from the

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mohammadmaso/kherad](https://github.com/mohammadmaso/kherad) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
