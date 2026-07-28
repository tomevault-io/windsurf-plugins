---
trigger: always_on
description: **Ghostwriter** is an offensive security operations platform for report writing, asset tracking, and assessment management. Built with Python 3.10, Django 4.2, TypeScript/React in Docker containers. ~484 Python files, ~76 JS/TS files, 77MB codebase.
---

# Ghostwriter Copilot Instructions

## Repository Overview

**Ghostwriter** is an offensive security operations platform for report writing, asset tracking, and assessment management. Built with Python 3.10, Django 4.2, TypeScript/React in Docker containers. ~484 Python files, ~76 JS/TS files, 77MB codebase.

**Stack:** Django 4.2 | Python 3.10 | PostgreSQL 16.4 | Redis 6 | React/TypeScript | Vite | Hasura GraphQL v2.39.1 | Channels 4.0 WebSockets | Django-Q2 background tasks | Docker Compose | Jinja2/python-docx/python-pptx for reports

## Environment Setup

**Prerequisites:** Docker 28.0.4+, Docker Compose v2.38.2+, pre-built `ghostwriter-cli-linux` in repo root

**⚠️ CRITICAL:** Installation requires internet access to Alpine mirrors. Sandboxed environments will fail with "network error" - this is a known limitation.

**Setup (5-10 min first run):**
```bash
chmod +x ghostwriter-cli-linux
./ghostwriter-cli-linux install --dev  # Builds images, initializes DB
./ghostwriter-cli-linux up --dev       # Start (or: docker compose -f local.yml up -d)
./ghostwriter-cli-linux down --dev     # Stop (or: docker compose -f local.yml down)
```

**Container commands:** `containers up/down/restart/build --dev`, `running` (list services)

## Build, Test, and Development

### Live Development
Django auto-reloads with `local.yml` - **NO rebuild needed** for Python/template/static changes.

**Rebuild required for:** New dependencies in `requirements/*.txt`, new tasks in `tasks.py`, Dockerfile changes
```bash
docker compose -f local.yml stop && docker compose -f local.yml rm -f && docker compose -f local.yml build && docker compose -f local.yml up -d
```

### Migrations
**ALWAYS run after model changes:**
```bash
docker compose -f local.yml run --rm django python manage.py makemigrations
docker compose -f local.yml run --rm django python manage.py migrate
```

### Tests (~45-60s for full suite)
```bash
docker compose -f local.yml run django coverage run manage.py test  # All tests
docker compose -f local.yml run django coverage run manage.py test ghostwriter.rolodex.tests  # Specific app
./ghostwriter-cli-linux test --dev  # Using CLI
docker compose -f local.yml run django coverage report -m  # Coverage report
```
**Note:** Tests include intentional errors. Success = "OK" at end, no "FAILED".

### Linting & Code Quality
**Python:** Black (line length 90), isort (profile=black), flake8 (max 240/.pylintrc, 120/setup.cfg), pylint-django
```bash
docker compose -f local.yml run django black .
docker compose -f local.yml run django isort .
docker compose -f local.yml run django flake8
```

**JS/TS:** Prettier, TypeScript strict
```bash
cd javascript && npm install  # First time only
npm run check && npm run format  # Type check + format
npm run build-frontend-prod     # Production build
```

## Project Structure

**Root:** `manage.py`, `local.yml`/`production.yml` (Docker Compose), `.env`, `pytest.ini`, `setup.cfg`, `.pylintrc`, `.isort.cfg`, `.coveragerc`, `ghostwriter-cli-*`

**Django Apps** (`ghostwriter/`, each with models/views/urls/templates/tests):
- `reporting/` (1.7MB) - Report generation, templates, findings
- `rolodex/` (836KB) - Client/project management
- `shepherd/` (780KB) - Infrastructure/domain management
- `api/` (312KB) - REST API, GraphQL handlers
- `commandcenter/` (308KB) - Dashboard/statistics
- `oplog/` (272KB) - Activity logging
- `home/` (168KB) - Landing/user dashboard
- `users/` (168KB) - Auth/user management
- `modules/` (380KB) - Shared utilities
- `singleton/` (64KB) - Global config
- `status/` (36KB) - Health checks
- `static/` (9.9MB), `templates/` (300KB), `factories.py` (36KB)

**Config:** `config/settings/{base,local,production,test}.py`, `urls.py`, `asgi.py` (WebSockets), `wsgi.py`

**Frontend:** `javascript/src/` (TypeScript/React), `package.json`, `vite.config.*.ts`, `tsconfig.json`

**Docker:** `compose/local/` (Django/Node Dockerfiles), `compose/production/`

**GraphQL:** `hasura-docker/metadata/` (tables, actions, permissions)

## CI/CD

**GitHub Actions** (`.github/workflows/`):
1. **workflow.yml** - Fast PR CI: Builds the cached Django image, runs tests against the minimal `ci.yml` stack, and uploads coverage to Codecov
2. **cli-install.yml** - Installer CI: Smoke-tests the full CLI installation for relevant PRs, then runs the full suite on `master`, nightly, and manual runs
3. **codeql-analysis.yml** - Security scan (Python/JS, push/PR/weekly Thu 6:31 AM)
4. **update-version.yml** - Version updates
5. **inactive-issues.yml** - Issue automation

**No pre-commit hooks.** Quality enforced via: manual linting → CI checks → code review

## Common Issues

1. **Container "stuck":** Django won't restart → `docker compose -f local.yml up -d`
2. **Alpine install fails:** "network error" → Known limitation, needs full internet access
3. **ImportError after adding deps:** → Rebuild containers (see above)
4. **Missing columns/tables:** → Run migrations (see above)
5. **Static files not loading:** → `docker compose -f local.yml run django python manage.py collectstatic --noinput`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GhostManager/Ghostwriter](https://github.com/GhostManager/Ghostwriter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
