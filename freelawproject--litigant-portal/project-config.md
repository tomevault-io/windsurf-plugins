---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Access to justice portal for self-represented litigants. Django 5.2 with server-rendered components (Django Cotton), Tailwind CSS v4, and Alpine.js for reactivity.

## Current Focus: Beta Demo — Housing Eviction Flow

Building a complete eviction flow from discovery to resolution for court partner demos. One topic, end-to-end, at production quality — every button/link does something, no placeholders. Court-neutral information where partner-specific data isn't available yet.

- [Milestone](https://github.com/freelawproject/litigant-portal/milestone/3) - Beta Demo: Housing Eviction Flow
- [Legal Flow](docs/overview-mapped-legal-flow.md) - Generic 9-stage flow (Triage / Prepare / Resolve); legal review artifact
- [Happy Path Narrative](docs/happy-path-jane.md) - Full AI · Auth end-to-end story (base for all variations)
- [Demo Flow](docs/demo-flow-jane.md) - Jane's 8-step demo flow (abbreviated)
- [User Flows Matrix](docs/user-flows.md) - 3×2 matrix (Full AI / Hybrid / Basic × Anon / Auth)
- [Retro Notes](docs/itc-demo-retro.md) - Lessons learned from ITC demo (Jan 2026)

## Environment Philosophy

Keep configuration **simple and consistent** across dev, CI/CD, and QA. Docker everywhere.

| Environment | Chat Provider | Config Source                              |
| ----------- | ------------- | ------------------------------------------ |
| Local dev   | OpenAI        | docker-compose.yml + `.env` (secrets only) |
| CI/CD       | None (mocked) | tox.ini - tests mock all providers         |
| QA/Staging  | OpenAI        | docker-compose prod profile + secrets/     |
| Production  | OpenAI        | docker-compose.yml + `.env` + secrets/     |

**QA environment:** `https://qa.litigantportal.com` — auto-deploys on merge to `qa` via GitHub Actions CD workflow. See `docs/QA-DEPLOY.md` for server setup. Uses the same docker-compose prod profile on a DigitalOcean VPS.

**Local dev setup:**

```bash
cp .env.example .env        # Add your OPENAI_API_KEY
make docker-dev             # Start dev environment
```

Chat model is configurable via `CHAT_MODEL` env var (LiteLLM format, e.g. `openai/gpt-4o-mini`).

## Commands

**`make lint`, `make test`, `make pre-commit`** — sandbox restrictions prevent Claude from running them. Always ask the user to run these manually. Never attempt to run them yourself.

### Local Development (Docker)

```sh
cp .env.example .env        # Add your OPENAI_API_KEY
make docker-dev             # Start dev environment
make docker-shell           # Shell into container
make docker-down            # Stop containers
```

**Dev URL:** `http://localhost` (or `http://portal.localhost`). Caddy runs on port 80 — **not** `:8000`. The `:8000` you'll see in `docker-compose.yml:101` is the container-internal gunicorn port, behind Caddy.

### Testing & Linting

```sh
make test                   # Auto-detects Docker: full suite if running, fast suite otherwise
make test fast              # Pass extra tox args through make
make -- test -e fast -- -k "ReadSecretTests"
make lint                   # Lint and format all code (via pre-commit)
```

`make test ...` forwards extra positional args to `tox` when running in Docker. Args that start with `-` are parsed by `make` itself, so use `make -- test ...` when passing tox or pytest flags.

### Direct Python commands (use .venv/bin/python)

For Django management commands outside Docker:

```sh
SECRET_KEY=dev .venv/bin/python manage.py check
SECRET_KEY=dev .venv/bin/python manage.py makemigrations
SECRET_KEY=dev .venv/bin/python manage.py migrate
SECRET_KEY=dev .venv/bin/python manage.py shell

# Run non-postgres tests locally (no Docker needed)
tox -e fast
# Run a specific test class
tox -e fast -- -k "ReadSecretTests"
```

## Pre-commit Hooks

Pre-commit runs automatically on commit. Key hooks:

- **ruff** - Python linting/formatting
- **djlint** - HTML template linting (errors only, no auto-formatting)
- **prettier** - JS, JSON, CSS, Markdown, YAML formatting
- **csp-inline-check** - Blocks inline event handlers (CSP compliance)

Run all hooks manually: `pre-commit run --all-files`

**Note:** djlint runs in **lint-only mode** (no auto-formatter). Its formatter was mangling template tags inside HTML attributes — expanding single-line `{% block %}` tags to multi-line and injecting whitespace into rendered attributes. See "Template Formatting" below for the manual conventions that replace the formatter.

### Before Committing

Always run before commits (especially after rebases or batch edits):

```bash
make pre-commit   # lint → test, short-circuits if lint fails/fixes anything
```

Equivalent to `make lint && make test`. If lint auto-fixes files, the target stops before tests — re-stage the changes and re-run. The name mirrors the `pre-commit` hook tool intentionally — same concept, different invocation surface.

### Template Formatting (Manual Conventions)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [freelawproject/litigant-portal](https://github.com/freelawproject/litigant-portal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-23 -->
