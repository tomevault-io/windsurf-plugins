---
trigger: always_on
description: Production deployment requirements — Docker, DigitalOcean, Vercel, CORS, pool, auth, observability
---


# Deployment Hardening

Deviations are bugs, not preferences.

## Deployment targets

Primary: **DigitalOcean** (or equivalent API host for the backend) + **Vercel** (frontend) + **Supabase** (auth + Postgres).

Optional: `docker-compose.yml` runs **Redis + backend** locally for container smoke checks. Database is always Supabase (local or hosted). It is not the production topology.

There are three environments: `development`, `test`, `production`. There is no staging.

## Backend host (production)

- Ship the same `backend/Dockerfile` image; the host sets listen `PORT` (e.g. `${PORT:-8000}` in CMD).
- Postgres uses Supavisor **session pooler** (port 5432, IPv4) in production. Direct connections (`db.xxx.supabase.co`) resolve to IPv6 only, which DO App Platform cannot route. Session mode preserves per-connection state and supports asyncpg prepared statements. Do NOT use the transaction pooler (port 6543) - asyncpg prepared statements are incompatible with transaction pooling. `PUBLIC_DB_USER=postgres.<project-ref>`, `PUBLIC_DB_HOST=aws-0-<region>.pooler.supabase.com`, `PUBLIC_DB_PORT=5432`, `PUBLIC_DB_SSL_MODE=require` are committed in `backend/.env.production`. `PRIVATE_DB_PASSWORD` comes from secrets. A single `PRIVATE_DATABASE_URL` secret is still supported; config normalizes it to `postgresql+asyncpg://`.
- `PRIVATE_JWT_SECRET` must be the Supabase project's JWT secret — not a random value.
- `PUBLIC_CORS_ORIGINS` must include all stable Vercel domains. `PUBLIC_CORS_ORIGIN_REGEX` can match preview deploys.
- `PUBLIC_FRONTEND_URL` needed for Xero OAuth redirects.
- Health check path used in CI smoke tests: `GET /api/beta/shared/health`.

## Vercel

- **Single** root `vercel.json` at repo root controls the Vercel project (build `cd frontend && pnpm install --frozen-lockfile && pnpm run build`, output, CSP, `git.deploymentEnabled`). No second `frontend/vercel.json` (avoids drift). `VERCEL_ORG_ID` / `VERCEL_PROJECT_ID` are **not** committed in `vercel.json`; use GitHub Environment secrets or gitignored `.vercel/project.json` after `vercel link`.
- Output: `frontend/dist`. SPA rewrite: all routes → `index.html`.
- Three `VITE_*` env vars are build-time (baked into JS). Source of truth for production: committed `frontend/.env.production` (loaded by Vite when `pnpm run build` runs in mode production). Changing requires redeploy.
- **Git vs Actions:** Repo sets `git.deploymentEnabled` with the production branch name (e.g. `main: false`) so Vercel’s [GitHub integration](https://vercel.com/docs/git/vercel-for-github) does not auto-deploy that branch while GitHub Actions runs `pull` + `build --prod --yes` + `deploy --prebuilt --prod --yes` (see [GitHub Actions + Vercel](https://vercel.com/docs/git/vercel-for-github#using-github-actions)). The key must match the **Production Branch** in Vercel project settings. Pin CLI with `npx vercel@50` under pixi. Deploy job attaches `--meta` (`githubCommitSha`, `githubCommitRef`, `githubRepo`, `githubActionsRunId`) for filtering in `vercel list --meta`. Environment secrets: `frontend_vercel_deployment` with `VERCEL_TOKEN`, `VERCEL_ORG_ID`, `VERCEL_PROJECT_ID`.
- Optional: Vercel can emit [`repository_dispatch`](https://vercel.com/docs/git/vercel-for-github#repository-dispatch-events) to GitHub (e.g. `vercel.deployment.success`) for post-deploy E2E; prefer that over `deployment_status` if you disable deployment notifications in Vercel Git settings.
- Security headers set in `vercel.json`: `X-Content-Type-Options`, `X-Frame-Options`, `Referrer-Policy`, `Permissions-Policy`.
- Immutable cache headers on `/assets/`.
- Build-time env template: `frontend/.env.example`.

## Docker

- Two-stage build: `builder` (compilers, uv sync) → `runtime` (venv only, no headers)
- uv is pinned via `COPY --from=ghcr.io/astral-sh/uv:0.7` in the builder stage
- Non-root: `appuser:appgroup` uid/gid 1001, `USER appuser` before `CMD`
- `ENV PYTHONUNBUFFERED=1 PYTHONDONTWRITEBYTECODE=1` in runtime stage — required for log streaming
- `CMD` uses gunicorn with `uvicorn.workers.UvicornWorker` for process management. Pass `--worker-tmp-dir /dev/shm` - DigitalOcean App Platform's container runtime can break gunicorn's default worker temp dir ([Dockerfile build reference](https://docs.digitalocean.com/products/app-platform/reference/dockerfile/)). No `uv run` wrapper in production.
- `devtools/` never copied into image

## CI/CD

- GitHub Actions **CI** (`.github/workflows/ci.yml`): standalone on **push to `dev`**, all **pull requests** to `main`/`dev`, **`workflow_dispatch`**, and **`workflow_call`** from **CD**. On **`main` pushes** there is **no** standalone CI - only the CD workflow runs and calls `ci.yml` as the CI gate (avoids duplicate test runs).
- **`dorny/paths-filter@v4`** drives **all four CI jobs**. Backend lint and test run when `backend/**` or `supabase/**` changed. Frontend lint runs when `frontend/**` changed. Frontend test runs when backend, supabase, **or** frontend changed (cross-stack regressions). If nothing in those dirs changed, all jobs skip.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [albusOS/sku-ops](https://github.com/albusOS/sku-ops) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-30 -->
