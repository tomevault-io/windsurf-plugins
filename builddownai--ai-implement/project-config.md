---
trigger: always_on
description: A Node.js service that polls Linear for issues labeled "AI-Implement" and dispatches GitHub Actions workflows that run Claude Code to implement them. It also provides an admin UI and manages workflow templates synced to target repos.
---

# AI-Implement — Codebase Guide

## What this is

A Node.js service that polls Linear for issues labeled "AI-Implement" and dispatches GitHub Actions workflows that run Claude Code to implement them. It also provides an admin UI and manages workflow templates synced to target repos.

## Architecture

```
Linear (AI-Implement label)
    ↓ poll every 60s (src/index.ts)
Node.js service on Fly.io
    ↓ workflow_dispatch (src/github.ts)
GitHub Actions in target repos (.github/workflows/claude-implement.yml)
    ↓ anthropics/claude-code-action
PR created → gap analysis posted → Linear updated
    ↓ /ai-implement comment (comment-trigger.yml)
Gap-fill run on existing PR
```

## Project structure

```
src/
  index.ts          — main entry: polling loop + HTTP server
  linear.ts         — Linear GraphQL client
  github.ts         — GitHub workflow_dispatch
  notify.ts         — notification adapter (slack | teams)
  config.ts         — SQLite-backed team→repo mappings
  dedup.ts          — SQLite deduplication + DB singleton
  poll-selection.ts — per-team capacity selection logic
  log.ts            — dispatch audit log (SQLite)
  admin.ts          — admin HTTP API (auth + CRUD)
  admin-html.ts     — re-exports the assembled admin HTML from admin-ui/index.ts
  admin-ui/         — string-composed admin SPA (see "admin-ui" section below)
  __tests__/        — Vitest unit tests

workflows/          — templates synced to target repos
  claude-implement.yml
  comment-trigger.yml
  claude-plan.yml   — planning workflow template (always synced)
  WORKFLOW.md       — Claude implementation prompt template (seeded once)
  PLANNING.md       — Claude planning prompt template (seeded once)

clients/            — one .toml per deployed client
  example-client.toml  — copy this to onboard a new client

scripts/
  provision-client.sh  — interactive client onboarding helper

.github/workflows/
  deploy-clients.yml — matrix deploy to all clients on push to main
  sync-workflow.yml  — sync workflow templates to target repos
  claude-review.yml  — Claude reviews PRs (auto for same-repo, /claude-review for forks)
  build-runner.yml   — build and push the session runner image to GHCR
```

## Running locally

```bash
cp .env.example .env   # fill in LINEAR_API_KEY, GITHUB_PAT
npm install
npm run dev            # runs src/index.ts via tsx
npm run dev:local      # rebuilds local session image, then runs local Docker jobs
```

Health check: `curl http://localhost:8080/`
Admin UI: `http://localhost:8080/admin` (requires ADMIN_ACCESS_CODE)

## Running tests

```bash
npm test              # vitest run (all tests)
npm run test:watch    # watch mode
npm run typecheck     # tsc --noEmit
```

## SQLite databases

All tables live in a single SQLite file at `DEDUP_DB_PATH` (default `/data/dedup.sqlite` in production, `./dedup.sqlite` locally).

| Table | Purpose |
|-------|---------|
| `dispatched` | Dedup — issue IDs dispatched in the last 24h |
| `mappings` | Team key → GitHub repo config |
| `dispatch_log` | Audit log, last 500 dispatches |

`dedup.ts` owns the DB singleton (`getDb()`). All other modules import `getDb` from `dedup.ts`.

## Key environment variables

| Variable | Required | Description |
|----------|----------|-------------|
| `LINEAR_API_KEY` | Yes | Linear personal API key |
| `GITHUB_APP_ID` | Yes | GitHub App numeric ID |
| `GITHUB_APP_PRIVATE_KEY` | Yes | GitHub App RSA private key (PEM, `\n`-escaped) |
| `NOTIFY_TYPE` | No | `slack` (default) or `teams` |
| `NOTIFY_WEBHOOK_URL` | No | Webhook URL; notifications skipped if unset |
| `ADMIN_ACCESS_CODE` | No | Admin UI password; UI disabled if unset |
| `DEDUP_DB_PATH` | No | SQLite path (default `/data/dedup.sqlite`) |
| `POLL_INTERVAL_MS` | No | Poll interval ms (default `60000`) |
| `PORT` | No | HTTP port (default `8080`) |

## Adding a new target repo

1. Add the team→repo mapping in the admin UI at `/admin`
2. Install the GitHub App on the target repo
3. Click **Sync workflows** on that project row — the orchestrator opens or updates a PR in the target repo with workflow files and starter prompt templates
4. Merge the PR in the target repo
5. Enable "Allow GitHub Actions to create and approve pull requests" in the target repo settings

The GitHub App must have **workflows** permission in addition to **contents** permission. GitHub rejects writes under `.github/workflows/` without it, so the Sync workflows action will fail before opening or updating the target-repo PR.

## admin-ui

The admin SPA at `/admin` is composed from string-exporting modules under `src/admin-ui/`. `src/admin-html.ts` is a thin re-export of `src/admin-ui/index.ts`. There is **no client-side build step** — all client JS is concatenated into a single inline `<script>` block at module load time.

| Module | Owns |
|---|---|
| `tokens.ts` | CSS custom properties (light + dark + accent + spacing + type + radius + shadow) |
| `components.ts` | All component classes (`.card`, `.tbl`, `.btn`, `.badge`, `.kpi`, `.alert`, `.drawer`, `.modal-card`, etc.) |
| `icons.ts` | SVG icon registry + `icon(name, size)` helper |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [BuildDownAI/AI-Implement](https://github.com/BuildDownAI/AI-Implement) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-19 -->
