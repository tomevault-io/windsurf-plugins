---
trigger: always_on
description: This file helps two kinds of agents work on ExcaliDash.
---

# AGENTS.md

This file helps two kinds of agents work on ExcaliDash.

## Project summary (what is ExcaliDash?)

ExcaliDash is a self-hosted dashboard and organizer for Excalidraw drawings, with persistent storage and live collaboration.
Core user-facing features include organizing drawings into collections, search, export/import for backup, and configurable authentication (local and optional OIDC).

## Helpers (Operations)

- Official supported deployment flow (production): `docker-compose.prod.yml`
  - `curl -OL https://raw.githubusercontent.com/ZimengXiong/ExcaliDash/main/docker-compose.prod.yml`
  - `docker compose -f docker-compose.prod.yml pull`
  - `docker compose -f docker-compose.prod.yml up -d`
- Production hardening quick checklist:
  - Pin images to a specific release tag (or digest) instead of `:latest` for reproducible upgrades/rollbacks.
  - Stable tags are published as `zimengxiong/excalidash-backend:<VERSION>` and `zimengxiong/excalidash-frontend:<VERSION>` (and also `:latest`).
  - Pre-release tags are published as `:dev` and `:<VERSION>-dev` (and do not update `:latest`).
  - Set fixed `JWT_SECRET` and `CSRF_SECRET` for portability and multi-instance/redeploy scenarios.
  - Set `FRONTEND_URL` to your public URL(s) and keep `TRUST_PROXY=false` unless you are behind a trusted proxy hop.
  - Ensure the backend volume is backed up (SQLite DB + persisted secrets).
- Re-check production health:
  - `docker compose -f docker-compose.prod.yml ps`
  - `docker compose -f docker-compose.prod.yml logs backend --tail=200`
  - `docker compose -f docker-compose.prod.yml logs -f backend`
- Upgrade production:
  - `docker compose -f docker-compose.prod.yml down` (if needed)
  - `docker compose -f docker-compose.prod.yml pull`
  - `docker compose -f docker-compose.prod.yml up -d`
- Check bootstrap/setup signal if onboarding blocks access:
  - `docker compose -f docker-compose.prod.yml logs backend --tail=200 | grep "BOOTSTRAP SETUP"`
- For local helper debugging with compose (repo checkout): `docker compose up -d` (uses `docker-compose.yml`; use `-f docker-compose.prod.yml` if you deployed from Docker Hub images)
- Must-read first for common user issues: `README.md`

### Pinning And Switching Docker Tags

Pin to a stable release (recommended for production):

```yaml
services:
  backend:
    image: zimengxiong/excalidash-backend:0.4.18
  frontend:
    image: zimengxiong/excalidash-frontend:0.4.18
```

Switch to pre-release images (rolling `:dev` tag):

```yaml
services:
  backend:
    image: zimengxiong/excalidash-backend:dev
  frontend:
    image: zimengxiong/excalidash-frontend:dev
```

Switch to a specific pre-release build (pinned `:<VERSION>-dev` tag):

```yaml
services:
  backend:
    image: zimengxiong/excalidash-backend:0.4.18-dev
  frontend:
    image: zimengxiong/excalidash-frontend:0.4.18-dev
```

Switch to a one-off custom dev tag (published by `make dev-release NAME=...`):

```yaml
services:
  backend:
    image: zimengxiong/excalidash-backend:0.4.18-dev-issue38
  frontend:
    image: zimengxiong/excalidash-frontend:0.4.18-dev-issue38
```

## Contributors (Code Changes)

- Architecture map: backend (`backend/src`), frontend (`frontend/src`)
- Config source of truth: `backend/src/config.ts` (env + validation)
- Entry server init: `backend/src/index.ts`
- API client: `frontend/src/api/index.ts`
- Realtime/editor behavior: `frontend/src/pages/Editor.tsx`
- Deployment flow: `docker-compose*.yml`, `backend/Dockerfile`, `frontend/Dockerfile`, entrypoint scripts
- Build/dev commands: `make build`, `npm test`, `make test-all`, `make test-e2e`
- Development contributor flow:
  - `make install` → `make dev`
  - stop with `make dev-stop`
  - `make dev-backend` / `make dev-frontend` if you only need one side

## Helper workflow

Prioritize operational steps, then point to the exact commands or files.

For setup and troubleshooting, start here.

- Goal check: confirm whether the user needs local dev, Docker Compose from Docker Hub images, locally built Docker, or E2E.
- Check whether the problem is already known:
  - If the agent has GitHub access, search issues for the exact error text/symptom and link the closest match.
  - If the agent cannot browse, ask the user to search and share the issue link (or paste issue text).
  - Note: `git` does not provide a native way to browse GitHub Issues. Use the web UI or GitHub CLI (`gh`).
  - Issue tracker: `https://github.com/ZimengXiong/ExcaliDash/issues`
  - Search tip: `https://github.com/ZimengXiong/ExcaliDash/issues?q=is%3Aissue+<paste+error+snippet>`
  - If `gh` is available:
    - Set repo: `gh repo set-default ZimengXiong/ExcaliDash`
    - Search/list: `gh issue list --search "<error snippet>"`
    - View: `gh issue view <number> -w`
    - Create (guided): `gh issue create`
    - If missing, prompt the user to install `gh` (recommended) or use the web UI.
- Use the `README.md` and `e2e/README.md` as the primary operator references.
- If the question is about one error, find the nearest environment or script path before proposing code changes.
- Encourage filing an issue when needed (new bug / unclear docs / missing troubleshooting):
  - Ask the user to include:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ZimengXiong/ExcaliDash](https://github.com/ZimengXiong/ExcaliDash) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
