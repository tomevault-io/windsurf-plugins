---
trigger: always_on
description: This file is the repository-level source of truth for day-to-day development workflow. Supporting detail lives in `docs/development/`, while executable command truth lives in `package.json`, `docs/Makefile`, and GitHub Actions workflows.
---

# InnoClaw Development Guidelines

This file is the repository-level source of truth for day-to-day development workflow. Supporting detail lives in `docs/development/`, while executable command truth lives in `package.json`, `docs/Makefile`, and GitHub Actions workflows.

## Source Of Truth

Use these files in this order when instructions conflict:

1. `AGENTS.md`
2. `package.json` scripts and `docs/Makefile`
3. `.github/workflows/*.yml`
4. Supporting narrative docs in `docs/development/`

## Supported Environment

- Node.js `24+` (`package.json` is authoritative; CI runs on Node 24 and 25)
- `npm` for dependency management and scripts
- Python `3.10+` for Sphinx documentation work (`docs/requirements.txt` currently pins `sphinx==8.1.3`)
- SQLite/Drizzle for local database migrations

## Repository Boundaries

Treat these directories as product code and documentation:

- `src/` application code
- `scripts/` repository scripts
- `drizzle/` database migrations
- `docs/` documentation sources
- `public/` shipped static assets
- `site/` docs homepage assets

Treat these directories as scratch space, references, or local-only artifacts. They are not part of routine app validation and must not be imported into shipped code:

- `reference/`
- `.worktrees/`
- `.superpowers/`
- `.venv-docs/`
- `batch-test-logs/`
- `test-results/`
- `tmp-playwright-review/`

## First-Time Setup

```bash
npm install
cp .env.example .env.local
mkdir -p data
npx drizzle-kit migrate
```

Then edit `.env.local` and set at least:

- `WORKSPACE_ROOTS`
- the API keys required for the provider(s) you plan to test

## Before You Start

- Run `git status --short` before large edits so you know whether you are entering a shared or dirty worktree.
- Identify the shared contracts your change may touch before coding. Common examples: schema, env vars, route response shapes, tool names, persisted session keys, and contributor-facing commands.
- Read the nearest source-of-truth files first: the relevant `package.json` script, nearby tests, and the matching page under `docs/development/`.
- Decide up front which validation commands and documentation updates your change will require. Do not wait until review time to discover missing follow-through.

## Daily Workflow

Start the app with either:

```bash
npm run dev
```

or the helper scripts:

```bash
bash dev-start.sh
bash dev-status.sh
bash dev-stop.sh
```

Use `npx tsc --noEmit` for a fast type-only pass when you do not need a full production build. The authoritative production verification remains `npm run build`.

## Validation Matrix

Run these checks before opening or updating a PR:

```bash
npm run lint
npm test
NEXT_TELEMETRY_DISABLED=1 npm run build
```

When documentation changes:

```bash
cd docs
make html
make html-zh
```

If your system `python3` is older than 3.10, create a dedicated docs virtualenv first:

```bash
python3.12 -m venv .venv-docs
.venv-docs/bin/python -m pip install -r docs/requirements.txt
```

When English docs change, refresh translations:

```bash
cd docs
make update-po
```

## Database Migration Rules

- Treat `src/lib/db/schema.ts` as the schema source of truth.
- When schema changes, generate and commit the matching migration under `drizzle/`.
- Apply migrations locally with `npx drizzle-kit migrate` before requesting review.
- Do not edit already-committed historical migrations unless you are intentionally fixing unreleased local work and understand the impact on existing databases.
- If a change touches seed/default skill data or migration helpers, review the affected files in `src/lib/db/` and verify the startup path that uses them.

## API Route Rules

- Keep `src/app/api/**/route.ts` focused on HTTP concerns: request parsing, auth/context checks, validation, status codes, and response shape.
- Move reusable or multi-step business logic into `src/lib/` or `src/core/`; avoid burying domain logic inside route handlers.
- Validate request params early and return explicit `4xx` responses for caller mistakes.
- For non-streaming JSON errors, prefer the shared helpers in `src/lib/api-errors.ts` when practical instead of duplicating response shapes.
- Add or update `route.test.ts` coverage when changing route behavior, validation, or error handling.

## Documentation Change Matrix

- If you change setup steps or local commands, update `AGENTS.md` and the relevant pages under `docs/development/`.
- If you add, rename, or remove environment variables, update `.env.example`, `docs/getting-started/environment-variables.md`, and any affected setup docs.
- If you change contributor workflow or validation expectations, update `CONTRIBUTING.md`, `docs/development/contributing.md`, and any README links that point contributors there.
- If you change architecture or directory boundaries, update `docs/development/project-structure.md`.
- If you change testing workflow, update `docs/development/testing.md`.
- If you change documentation workflow itself, update `docs/development/documentation.md`.

## Collaboration Rules


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SpectrAI-Initiative/InnoClaw](https://github.com/SpectrAI-Initiative/InnoClaw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-19 -->
