---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Documentation Guidelines

- **`.claude/notes/`** - Internal documentation and plans created by Claude to help with its work
- **`docs/`** - Documentation for humans (user guides, API docs, etc.)
- When creating analysis documents, optimization plans, or working notes, place them in `.claude/notes/`
- Only create user-facing documentation in `docs/` when explicitly requested

## Quick Reference (Most Important)

**Critical Commands:**

- Start dev server: `./scripts/dev.sh` (starts Django + CSS watch, per-worktree isolation)
- Format code: `./scripts/fmt.sh`
- Run tests: `pytest -n auto`
- Django commands: Use `manage` (not `python manage.py`)
- Production shell: `manage prodshell` (read-only access to production database)
- Don't commit CSS files - they're auto-generated from SCSS
- The virtualenv is auto-activated for every Bash command by a `SessionStart` hook
  (`scripts/activate_venv_hook.sh`) that persists `.venv` into `CLAUDE_ENV_FILE`.
  There is no need to prefix commands with `. .venv/bin/activate &&`.
- The session hook also sets `DB_NAME` and `DJANGO_PORT` per-worktree — `manage` and `pytest`
  automatically target the correct database.

**Key Principles:**

- Server-rendered HTML, not SPA
- **URL-driven UI state.** Any state that picks a form variant, switches a
  visible section, opens a modal, or selects a tab belongs in the URL
  (path or query string). The server renders the right variant. JS may
  enhance (live preview, async validation, autocomplete) but the page MUST
  work — and be linkable — without it. **Do not** mutate forms client-side
  to swap fields, swap mode choices, hide/show sections, or alter
  validation. If you reach for `addEventListener('change', …)` to rewrite
  a form, you've probably skipped a navigation. See the rationale and the
  full rule in `.claude/skills/gyrinx-conventions/SKILL.md`.
- Mobile-first design
- Look up model definitions before use - don't assume field names
- Always validate redirect URLs with `safe_redirect`

## Infrastructure

- All our infra is in GCP europe-west2 (London)
- In prod, the user uploads bucket name is gyrinx-app-bootstrap-uploads

## Local Development (Per-Worktree Isolation)

Each git worktree gets its own Postgres database and Django port, started with a single command.

- **Setup (once per machine):** `./scripts/setup-local-postgres.sh` — installs Postgres 16 + pgAdmin via Homebrew,
  migrates data from Docker
- **Start dev server:** `./scripts/dev.sh` — ensures DB exists (forks from template if needed), provisions a
  per-worktree `.venv` on first run in a child worktree, runs migrations, runs `npm install` if
  `node_modules` is missing/stale, does an initial `npm run css` build if `styles.css` is missing/stale,
  then starts Django runserver + npm watch. **Always confirm the `CSS ready:` / `CSS file:` lines appear
  in the startup output — `npm run watch` alone never produces an initial build, so without `dev.sh`
  doing the seed build you'd get unstyled pages.**
- **Reset a worktree DB:** `./scripts/dev.sh --reset-db` — drops and re-forks from template
- **Rebuild a worktree's venv:** `./scripts/dev.sh --reset-venv` — wipes and re-provisions `${WT_ROOT}/.venv`
- **Clean up orphans:** `./scripts/cleanup-worktree-dbs.sh` — drops orphan DBs + reports worktree `.venv` sizes

**How it works:**

- Main worktree uses `gyrinx_main` database (port 8000) — this is the template with curated test data
- Child worktrees get `gyrinx_wt_{hash}` databases forked via `CREATE DATABASE ... TEMPLATE`
- Ports are deterministic per worktree path (range 8100-9599)
- **Each child worktree gets its own `.venv` with `gyrinx` editable-installed from that worktree**, so
  `import gyrinx` always resolves to worktree-local code (new migrations, new models, etc.). Without this,
  `manage migrate` from a child worktree silently misses new migrations and `pytest` fails with
  `ImportError`. `./scripts/dev.sh` provisions the venv via `uv venv` + `uv pip install --editable .` on
  first run (~1 minute). Main worktree continues to use whatever venv it already had.
- The session hook (`activate_venv_hook.sh`) auto-sets `DB_NAME` and `DJANGO_PORT` for every
  Claude Code Bash invocation
- `setup-local-postgres.sh` appends a block to `.venv/bin/activate` so that
  `source .venv/bin/activate` from any interactive terminal also exports the
  per-worktree DB env vars. **Re-activate the venv after switching worktrees**
  — the hook reads `git rev-parse --show-toplevel` at activation time, not on
  every command. Without this, `pytest` and `manage` from a plain shell fall
  back to `settings.py` defaults (user=postgres) and fail with
  "role postgres does not exist".
- `setup-local-postgres.sh` also tunes `max_locks_per_transaction = 256` in
  the local cluster (the default 64 is too low for pytest-xdist with 12
  workers each running syncdb in parallel — symptom is "out of shared memory").
- pgAdmin 4 (local app) connects to localhost:5432 and is pre-registered with
  a "Gyrinx (local)" server on first setup (CLI-imported into pgAdmin's
  SQLite config at `~/.pgadmin/pgadmin4.db`)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gyrinx-app/gyrinx](https://github.com/gyrinx-app/gyrinx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-03 -->
