---
trigger: always_on
description: This file provides guidance to AI coding agents when working with code in this repository.
---

# AGENTS.md

This file provides guidance to AI coding agents when working with code in this repository.

## Project Overview

sbomify is a Software Bill of Materials (SBOM) and document management platform. It supports both CycloneDX and SPDX formats, vulnerability scanning, compliance assessments, and document artifact management.

**Key principle**: sbomify never modifies security artifacts (ADR-004). Artifacts are stored exactly as received — immutable. All analysis (vulnerability scanning, compliance checking) produces separate output without altering the original artifact.

## Build and Development Commands

### Setup and Running

```bash
# Start development environment with Docker (recommended)
./bin/developer_mode.sh build
./bin/developer_mode.sh up

# Alternative: Run Django locally with Docker services
docker compose up sbomify-db sbomify-minio sbomify-createbuckets -d
uv sync && bun install
uv run python manage.py migrate
uv run python manage.py runserver  # Terminal 1
bun run dev                         # Terminal 2 (Vite)
```

### Testing

Always run tests in Docker:

```bash
# Start test services
docker compose -f docker-compose.tests.yml up -d

# All tests (parallel — requires pytest-xdist installed in container)
docker compose -f docker-compose.tests.yml exec tests uv run pytest -n auto --ignore=sbomify/apps/core/tests/e2e

# All tests (sequential)
docker compose -f docker-compose.tests.yml exec tests uv run pytest --ignore=sbomify/apps/core/tests/e2e

# Specific file or directory
docker compose -f docker-compose.tests.yml exec tests uv run pytest sbomify/apps/sboms/tests/

# Single test with debugger
docker compose -f docker-compose.tests.yml exec tests uv run pytest --pdb -x -s sbomify/apps/sboms/tests/test_upload.py::test_name

# Coverage report (must be >= 80%)
docker compose -f docker-compose.tests.yml exec tests uv run coverage run -m pytest
docker compose -f docker-compose.tests.yml exec tests uv run coverage report
```

If tests fail with `database "test_sbomify_test" already exists` or `is being accessed by other users` (stale DB from killed parallel runs), clean up:

```bash
# Kill stale connections and drop test DB
docker compose -f docker-compose.tests.yml exec db psql -U sbomify_test -d postgres \
  -c "SELECT pg_terminate_backend(pid) FROM pg_stat_activity WHERE datname LIKE 'test%' AND pid <> pg_backend_pid();"
docker compose -f docker-compose.tests.yml exec db psql -U sbomify_test -d postgres \
  -c "DROP DATABASE IF EXISTS test_sbomify_test;"

# If connections persist, restart the DB container
docker compose -f docker-compose.tests.yml restart db
# Wait for it, then drop
docker compose -f docker-compose.tests.yml up -d
sleep 15
docker compose -f docker-compose.tests.yml exec db psql -U sbomify_test -d postgres \
  -c "DROP DATABASE IF EXISTS test_sbomify_test;"
```

E2E tests use Playwright via Chrome DevTools Protocol in Docker with visual regression (baseline screenshots in `__snapshots__/`, diffs in `__diffs__/`):

```bash
docker compose -f docker-compose.tests.yml exec tests uv run pytest sbomify/apps/core/tests/e2e/
```

Frontend tests:

```bash
bun test
bun test path/to/file.spec.ts
```

### Key Test Fixtures

Global fixtures (no import needed — registered in root `conftest.py`):

| Fixture                         | What it provides                                                            |
| ------------------------------- | --------------------------------------------------------------------------- |
| `sample_user`                   | Test user from `DJANGO_TEST_USER` env                                       |
| `guest_user`                    | Second standalone user (no team role assigned)                              |
| `sample_team`                   | Bare Team with no members                                                   |
| `sample_team_with_owner_member` | Owner `Member` for `sample_user` in `sample_team` (access team via `.team`) |
| `team_with_community_plan`      | Team + community billing plan                                               |
| `team_with_business_plan`       | Team + active business subscription                                         |
| `authenticated_api_client`      | `(Client, AccessToken)` tuple — use `get_api_headers(token)` for auth       |
| `authenticated_web_client`      | Django Client with full session for web tests                               |
| `ensure_billing_plans`          | Creates billing plan objects (use explicitly)                               |

Session setup helper: `setup_authenticated_client_session(client, team, user)` from `sbomify.apps.core.tests.shared_fixtures`.

Test settings: `sbomify.test_settings`. Tests run with `--nomigrations` (bare schema). Deselect slow tests: `-m "not slow"`.

### Linting and Formatting

```bash
# Python - ALWAYS run after changes
uv run ruff check . --fix
uv run ruff format .
uv run mypy .

# TypeScript/JavaScript
bun lint          # Check only
bun lint-fix      # Fix issues

# Django templates (Jinja2)
uv run djlint . --extension=j2 --check
uv run djlint . --extension=j2 --lint

# Run all pre-commit hooks
uv run pre-commit run --all-files
```

### Building

```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sbomify/sbomify](https://github.com/sbomify/sbomify) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
