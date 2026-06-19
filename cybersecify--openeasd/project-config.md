---
trigger: always_on
description: External Attack Surface Detection platform. Scans domains for network and
---

# CLAUDE.md — OpenEASD Django Project

External Attack Surface Detection platform. Scans domains for network and
web vulnerabilities using a dynamic workflow engine with auto-registered tools.

## GitHub Flow

**Rule:** Never commit to `main` directly.

**Branch prefixes** (only two):
- `feat/` — new features
- `fix/` — everything else (bugs, deps, config, refactor, docs, cleanup)

**Commit message prefixes** — use the most specific one that fits:

| Prefix | When to use |
|---|---|
| `feat:` | New user-facing feature |
| `fix:` | Bug fix |
| `docs:` | Documentation only (README, CHANGELOG, CONTRIBUTING, CLAUDE.md) |
| `ci:` | GitHub Actions, CI config, Dockerfile, dependabot |
| `chore:` | Deps, tooling, config — no behavior change |
| `test:` | Tests only — no production code change |

Branch prefix maps to commit prefix: `feat/` → `feat:`, `fix/` → any of the above.

### Steps for every task

1. **Sync main:**
```bash
git checkout main && git pull
```
   If `git pull` complains about uncommitted changes or a dirty working tree, stop and investigate before continuing — don't stash blindly, you may have unpushed work from a previous branch.

2. **Create branch:**
```bash
git checkout -b feat/short-descriptive-name
# or
git checkout -b fix/short-descriptive-name
```

3. **Work and commit** with `feat:` or `fix:` prefixed messages:
```bash
git commit -m "feat: add opening accuracy skill"
git commit -m "fix: guard empty games list in watcher"
```

4. **Open PR:**
```bash
gh pr create --title "..." --body "..."
```

5. **Squash-merge and delete remote branch:**
```bash
gh pr merge --squash --delete-branch
```

6. **Return to main and sync:**
```bash
git checkout main && git pull
```

7. **Delete local branch:**
```bash
git branch -D feat/your-branch-name
```
   Use `-D` (capital), not `-d`. After a squash merge, the squashed commit on `main` has a different SHA than your branch's commits, so `git branch -d` will refuse with "not fully merged" even though the PR is merged. `-D` force-deletes, which is safe here because the PR merge is the source of truth.

   Optionally, prune stale remote-tracking refs:
```bash
git fetch --prune
```

### Tagging

Tag `main` at meaningful milestones (not every PR). Use semantic versioning:
- `feat/` work → bump **minor** (v0.6.0 → v0.7.0)
- `fix/` work only → bump **patch** (v0.6.0 → v0.6.1)

```bash
git tag v0.7.0
git push origin v0.7.0
```

Check the latest tag anytime with:
```bash
git describe --tags --abbrev=0
```

## CI/CD (GitHub Actions)
- Pipeline: `.github/workflows/ci.yml` — runs on every push to `main` and `v*` tags
- **4 parallel jobs:**
  - `test` — pytest (fast, excludes `test_domain_security.py`), bandit (SAST), pip-audit (CVE scan)
  - `frontend` — `npm ci && npm run build`
  - `docker` — `docker buildx build` for `linux/amd64` (no push, cache check)
  - `publish` — builds `linux/amd64` + `linux/arm64` and pushes to `ghcr.io/cybersecify/openeasd`
- **Publish triggers:** every push to `main` (`:latest` tag) and `v*` tags (`:vX.Y` tag)
- Runner: `ubuntu-24.04`, Python 3.12, `uv sync --group dev` for deps, `libcairo2-dev gcc` system deps required
- `pip-audit --ignore-vuln PYSEC-2025-183` — disputed PyJWT weak-key-length CVE, no fix available

## Commands
- Always use `uv run python` instead of `python` or `python3`
- Always use `uv run manage.py` for Django management commands (e.g. `uv run manage.py check`)
- Always use `uv run pytest` for running tests
- The slow `tests/unit/test_domain_security.py` (41 tests) makes real DNS/RDAP calls — exclude it for fast CI runs:
  `uv run pytest tests/ --ignore=tests/unit/test_domain_security.py`

## Stack

### Backend
- Django 5+ with plain Django views (no DRF, no Celery, no Redis)
- **Django Ninja** REST API under `/api/` — Schema-based, auto-docs at `/api/docs`
- **JWT Bearer auth** — access + refresh tokens via `djangorestframework-simplejwt` (ninja-jwt wrapper); token blacklist handled by simplejwt's built-in `OutstandingToken`/`BlacklistedToken` models
- **Django-Q2** — background task queue for scan execution AND all scheduling (ORM broker, tasks stored in Django DB). `setup_core_schedules()` in `SchedulerConfig.ready()` registers daily scans, stuck-scan watchdog, JWT token purge, and per-domain monitoring jobs as `django_q.models.Schedule` entries. APScheduler has been fully removed.
- **WhiteNoise** — serves collected static files (frontend bundle) when `DEBUG=False` (Docker/prod); uses `CompressedManifestStaticFilesStorage` for gzip + content-hash fingerprinting
- SQLite database (dev), configurable via `DB_NAME` env var

### Frontend (React SPA — new primary UI)
- **React 19 + Vite 8** — `frontend/` directory, builds to `frontend/dist/`
- **shadcn/ui** — component library on top of Tailwind CSS 3 + Radix UI; CSS variables in `src/index.css`; components in `src/components/ui/`
- Vanilla popstate-based router in `App.jsx` (no react-router)
- JWT `apiFetch` in `src/api/client.js` — sends `Authorization: Bearer <token>` header; 401 clears tokens and redirects to `/login`
- `auth.js` — isolated localStorage helpers (`getToken`, `setTokens`, `clear`, `isLoggedIn`)
- `useFetch` / `usePolling` hooks for data fetching and live scan status (3s poll)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cybersecify/OpenEASD](https://github.com/cybersecify/OpenEASD) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-18 -->
