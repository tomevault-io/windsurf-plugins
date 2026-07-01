---
trigger: always_on
description: Kamu is a physical library management app (Django). It is being modernized from
---

# Kamu — AI Agent Guide

## Project Summary

Kamu is a physical library management app (Django). It is being modernized from
a React SPA + Django REST Framework architecture to server-rendered Django
templates with HTMX, petite-vue, and Tailwind CSS.

**Read these docs before starting any work:**
- `docs/ARCHITECTURE.md` — System design and principles
- `docs/TECH_STACK.md` — Current vs target technology
- `docs/TDD_WORKFLOW.md` — Mandatory TDD process
- `docs/FEATURES.md` — Feature breakdown and phase mapping
- `docs/USER_STORIES.md` — Acceptance criteria for every feature
- `docs/DJANGO_STRUCTURE.md` — Target project layout, URL patterns, view patterns
- `docs/FRONTEND.md` — HTMX, petite-vue, and template patterns
- `docs/DATABASE_SCHEMA.md` — Data model (no changes planned)
- `docs/DOCKER.md` — Podman-first container strategy
- `docs/ASSETS.md` — Tailwind CSS setup and static asset management

**Track all changes in:** `docs/AI_CHANGELOG.md`

---

## Current Status

**Active Phase:** COMPLETE
**Overall Progress:** All phases (0–4) COMPLETE. Modernization finished.

---

## Phase Overview

| Phase | Name                        | Status      | Description                                    |
|-------|-----------------------------|-------------|------------------------------------------------|
| 0     | Foundation & Infrastructure | COMPLETE    | Feature toggle, Podman, Postgres, CI, cleanup  |
| 1     | Asset Pipeline              | COMPLETE    | Tailwind CSS, HTMX/petite-vue vendor, base templates |
| 2     | Frontend Migration          | COMPLETE    | Page-by-page conversion to Django templates    |
| 3     | Legacy Cleanup              | COMPLETE    | Remove DRF, React, Webpack, Node.js            |
| 4     | Polish                      | COMPLETE    | Performance, accessibility, final docs         |

---

## Phase 0: Foundation & Infrastructure

### P0.1: Celery Feature Toggle [DONE 2026-03-19]

**Goal:** Gate all Celery/Redis functionality behind `KAMU_ENABLE_ASYNC_TASKS` env var.

**TDD Steps:**
1. Write test: `core/test/test_feature_toggles.py`
   - Test `run_async_task()` calls `.delay()` when toggle is True
   - Test `run_async_task()` is a no-op when toggle is False
2. Write test: `waitlist/test/test_tasks.py` (update existing)
   - Test book return skips notification when toggle is off
   - Test waitlist join skips notification when toggle is off
3. Run tests — confirm RED
4. Implement `core/feature_toggles.py`
5. Add `KAMU_ENABLE_ASYNC_TASKS` to `core/settings/common.py`
6. Update `books/models.py` and `waitlist/models.py` to use `run_async_task()`
7. Run tests — confirm GREEN
8. Update `.env` with `KAMU_ENABLE_ASYNC_TASKS=False`

**Files to modify:**
- `core/feature_toggles.py` (new)
- `core/settings/common.py`
- `books/models.py`
- `waitlist/models.py`
- `waitlist/tasks.py`
- `books/cron/send_notification.py`
- `core/test/test_feature_toggles.py` (new)
- `.env`

---

### P0.2: Podman + podman-compose [DONE 2026-03-19]

**Goal:** Replace Docker with Podman for all container operations.

**Steps:**
1. Rename `Dockerfile` → `Containerfile`
2. Create `podman-compose.yml` (replace `docker-compose.yml`)
3. Add PostgreSQL with healthcheck
4. Add optional Redis + Celery worker (behind `async` profile)
5. Update `Makefile` targets (remove `docker-` prefix)
6. Remove old `docker-compose.yml`

**Manual test for developer:**
- [ ] Run `podman-compose up` — web and database start
- [ ] Run `make migrate` — migrations succeed
- [ ] Run `make createsuperuser` — can create admin user
- [ ] Run `make loaddata` — seed data loads
- [ ] Visit http://localhost:8000 — app loads (React SPA still works)

---

### P0.3: PostgreSQL for All Environments [DONE 2026-03-19]

**Goal:** Eliminate SQLite. Use PostgreSQL everywhere.

**Steps:**
1. Update `core/settings/dev.py` to use `DATABASE_URL` defaulting to local Postgres
2. Update `core/settings/test.py` to use PostgreSQL
3. Remove SQLite database file from .gitignore (if tracked)
4. Verify all existing tests pass against PostgreSQL

**Manual test for developer:**
- [ ] Delete `db.sqlite3`
- [ ] Run `make migrate` → `make loaddata` → visit app
- [ ] Run `make test` — all tests pass

---

### P0.4: GitHub Actions CI [DONE 2026-03-19]

**Goal:** Replace CircleCI with GitHub Actions.

**Steps:**
1. Create `.github/workflows/cicd.yml`
   - Python tests with PostgreSQL service container
   - Coverage reporting
   - Linting (flake8)
2. Remove `.circleci/` directory
3. Remove `.codeclimate.yml`
4. Update `Makefile` to remove CodeClimate references

---

### P0.5: Remove Heroku/Dokku References [DONE 2026-03-19]

**Goal:** Clean up all deployment references to Heroku and Dokku.

**Files to modify/remove:**
- `Procfile` — remove
- `app.json` — remove
- `.buildpacks` — remove
- `.profile` — remove
- `.profile.d/` — remove
- `runtime.txt` — remove (Python version in pyproject.toml)
- `package.json` — remove `heroku-prebuild`, `heroku-postbuild` scripts
- `README.md` — remove Heroku/Dokku deployment sections

---

### P0.6: Python 3.12 Compatibility Check [DONE 2026-03-25]

**Goal:** Test if Okta SAML2 plugin works with Python 3.12.

**Research findings (2026-03-19):**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ayr-ton/kamu-oss](https://github.com/ayr-ton/kamu-oss) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
