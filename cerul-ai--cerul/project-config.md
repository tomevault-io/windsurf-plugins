---
trigger: always_on
description: When collaborating in issues, reviews, or direct agent/user conversations, reply to the repository owner in Chinese by default.
---

# Repository Guidelines

## Communication & Language
When collaborating in issues, reviews, or direct agent/user conversations, reply to the repository owner in Chinese by default.

Unless explicitly requested otherwise:

- code should be written in English
- public-facing docs should be written in English
- `README.md` content should be written in English
- identifiers, comments, commit messages, and API field names should be written in English

If a document is clearly internal-only and not intended for the public repository, Chinese is acceptable when that is more practical.

## Project Principles
Cerul is an open-core project. The codebase is public; the core moat is not.

Treat these as baseline project rules:

1. Data, indexes, prompts, internal evaluation assets, and model weights are not part of the public repository.
2. Public code should stay reusable and infrastructure-oriented.
3. Avoid adding stack changes unless there is a clear project-level decision to do so.
4. Prefer simple, maintainable building blocks over framework churn.
5. Keep the API layer thin and push heavy processing into workers.

## Product & Architecture Guardrails
Cerul exposes one unified product surface and routes internally by source and processing strategy.

This repository is the `cerul` web app after the backend split. Private backend code now lives in sibling repositories:

- `cerul-api` for the Hono / Cloudflare Workers API and database migrations
- `cerul-worker` for indexing, evaluation, and media-heavy processing

Default architectural assumptions:

- frontend: Next.js
- api: Hono on Cloudflare Workers
- database: Neon PostgreSQL with pgvector
- auth: Better Auth
- heavy indexing and media processing: Python workers
- first agent integration path: installable skill over direct HTTP API

Do not introduce a second primary backend stack, ORM stack, or deployment platform without an explicit decision. In particular, do not casually pivot the project toward TanStack Start, Cloudflare D1, or Drizzle as the default foundation.

For agent integrations, keep the first phase simple:

- ship a documented HTTP API
- ship a skill that uses API keys
- do not add an MCP adapter unless there is clear external demand

Keep these boundaries intact:

- `cerul-api` handles request orchestration, auth, usage, and API responses
- `cerul-worker` handles indexing and other media-heavy processing
- shared worker runtime helpers live in `cerul-worker/workers/common/`
- frontend pages should not become the primary business logic layer

Worker-side indexing should continue to follow a shared step-pipeline approach:

- keep indexing flows step-based and composable
- prefer shared context over ad hoc cross-step state passing
- keep idempotency in mind for every pipeline step
- avoid embedding heavy media logic directly inside API handlers

## Open Source Boundary
Assume everything inside this repository may become public.

Do not commit:

- internal strategy memos
- fundraising materials
- internal prompts
- internal benchmark sets
- production data exports
- proprietary ranking parameters
- secrets, local dumps, or ad hoc research files

If material is useful internally but not suitable for the repository, keep it under the local private workspace rather than adding it here.

## Project Structure & Module Organization
This repository is the public-safe server-side Next.js app. Put the web app in `frontend/` (`app/`, `components/`, `lib/`), keep public-safe docs in `docs/`, installable agent skills in `skills/`, the public API contract copy in `openapi.yaml`, and frontend-side automation in `scripts/`. Do not reintroduce `api/`, `workers/`, `db/`, or `config/` as top-level product directories here; those belong in the private sibling repositories.

Do not create a top-level `sdk/` just to wrap Cerul's own backend calls. An SDK only belongs in the repo once there is a real public client package to ship and version independently. Until then, frontend code should call backend APIs directly, and agent integrations should prefer a documented skill plus direct HTTP access. Treat MCP the same way: it is a future adapter, not a required first-class module in the initial repository layout.

## Build, Test, and Development Commands
This repository is still scaffold-first: no root `package.json`, `pyproject.toml`, or `Makefile` is committed yet. Today, the main setup command is:

```sh
cp .env.example .env
```

Use it to seed local secrets, runtime profile selection, and any optional env overrides before running new app code. Frontend browser code must consume a derived public config subset rather than reading raw repo config files directly. When you add runnable modules, expose explicit commands close to that module and document them in both `README.md` and this file.

Current frontend commands:

```sh
pnpm --dir frontend install
pnpm --dir frontend dev
pnpm --dir frontend lint
pnpm --dir frontend test
pnpm --dir frontend build
```

Repository-level reset:

```sh
./rebuild.sh
./rebuild.sh --fast
```

## Coding Style & Naming Conventions

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cerul-ai/cerul](https://github.com/cerul-ai/cerul) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
