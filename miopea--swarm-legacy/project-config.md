---
trigger: always_on
description: > See `~/.claude/CLAUDE.md` for universal rules (code quality, verification, shipping vocabulary, secrets, swarm) and its routing map to `rcg-architecture/docs/standards/`.
---

# Swarm (legacy) — Project Guide

> See `~/.claude/CLAUDE.md` for universal rules (code quality, verification, shipping vocabulary, secrets, swarm) and its routing map to `rcg-architecture/docs/standards/`.

## Orientation

- **What this is.** `swarm-ai` — **Swarm (legacy)**, a hive-mind orchestrator for
  Claude Code agents. Python ≥3.12, aiohttp daemon plus PTY workers, one `swarm`
  entry point. Superseded by Swarm Next; this repo is maintenance-only. The
  GitHub repo is `miopea/swarm-legacy` (renamed from `miopea/swarm`).
  **Display strings say "Swarm (legacy)"; the package name does not** — it is
  still `swarm-ai` on PyPI. The *installed* names do move, but only when the
  operator runs `swarm relocate`: `swarm` → `swarm-legacy`, `swarm.service` →
  `swarm-legacy.service`, `~/.swarm` → `~/.swarm-legacy`. Both entrypoints ship
  together so an un-relocated install keeps working.
- **How it runs here.** This box is **relocated**: systemd **user** unit
  `swarm-legacy.service` ("Swarm (legacy) Web Dashboard"),
  `ExecStart=…/swarm-legacy serve`, daemon API on **:9090**, state in
  `~/.swarm-legacy`. Not a system unit — `systemctl --user`, not
  `sudo systemctl`. Pre-relocation installs still use `swarm.service`.
- **`swarm-next` is a DIFFERENT system on the same box.**
  `swarm-next-api.service` and `swarm-next-terminal-host.service` run alongside
  this one from `~/projects/personal/swarm-next`. Restarting the wrong unit looks
  like a fix that changed nothing.
- **All state is one SQLite file, in the state dir** — `~/.swarm-legacy/swarm.db`
  after `swarm relocate`, `~/.swarm/swarm.db` before it (~99 MB). **Never
  hardcode either**: call `swarm.paths.state_dir()`, which prefers the relocated
  directory. Beside the DB, `invariant-log.jsonl` (Tier 0 hook decisions) and
  `pty-writes.jsonl`. Nothing authoritative lives in this repo's tree.
- **No deploy.** There is no production target and no `deploy.md`. Push is the
  terminal state — say `pushed`, never `shipped`.
- **CI gates on two Python versions**, `test (3.12)` and `test (3.13)`, and they
  are required checks on `main`. Both must be green before merge.

### Two `swarm.db` traps that have each cost a real investigation

Both look like a correct query returning a true zero.

- **`status` is `done`, NOT `completed`.** Querying `status='completed'` returns
  `0` from a `0` denominator, which reads exactly like "nothing has happened yet".
  Live values: `active`, `assigned`, `backlog`, `blocked`, `done`, `unassigned`.
- **`completed_at` is a unix FLOAT, not an ISO string.** A string comparison
  silently matches nothing and reports a clean zero.

Query it read-only — `sqlite3.connect(f"file:{db}?mode=ro", uri=True)` — and
print the denominator beside any count. Resolve `db` with
`swarm.paths.state_dir() / "swarm.db"`, never a hardcoded `~/.swarm`: after
`swarm relocate` that path is gone and the query reads nothing.

## Quick Reference

### Essential Rules
| Rule | Action |
|------|--------|
| Before commit | Use `/commit` slash command |
| Pre-commit validation | Use `/check` slash command |
| Bug fix | Use `/fix-and-ship` or `/diagnose` first |
| Test failures | STOP — fix before continuing |
| Warnings | STOP — warnings = failures |
| `type: ignore` | FORBIDDEN — fix the type error |
| Creating a file | SEARCH existing code first |
| Installed tool stale? | Follow *Dev vs Installed Version* in [`docs/project-notes.md`](docs/project-notes.md) — that is the canonical cache-busting reinstall |

### Key Files
| File | When to Check |
|------|---------------|
| `swarm.yaml` | Configuring workers, drones, queen, groups |
| `src/swarm/drones/state_tracker.py` | Debugging state detection issues (provider patterns in `src/swarm/providers/`) |
| `src/swarm/drones/pilot.py` | Understanding the poll loop and drone actions |
| `src/swarm/server/daemon.py` | Core daemon lifecycle, events, WebSocket broadcasts |
| `src/swarm/server/routes/` | HTTP/WebSocket endpoint handlers (tasks, workers, queen, jira, config, websocket, …) |
| `src/swarm/web/routes/` | Page + HTMX partial routes, login/passkeys, PWA manifest & share target |
| `src/swarm/server/api.py` | The aiohttp app factory — registers no routes itself; owns session auth, CSRF, security headers, and rate limiting |
| `src/swarm/web/templates/dashboard.html` | Dashboard markup (modals, panels, partial mount points) |
| `src/swarm/web/static/dashboard.js` | Dashboard behaviour — WebSocket wiring, terminal, task board, keyboard shortcuts |

---

## Design Principles

### Architecture Guidelines
- **Event-driven decoupling** — Pilot emits events, daemon subscribes; never tight-couple components
- **Feature-based modules** — Organize by domain (worker/, drones/, queen/, tasks/), not by layer
- **Async everywhere** — All PTY/holder calls are async; all I/O is async. Never block the event loop.
- **Thin API handlers** — Validation in handlers, business logic in daemon/pilot/managers

---

## Critical Rules

After making code edits, always run `uv run ruff format` before validation checks. Never commit unformatted code.

### Post-Change Validation (MANDATORY)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [miopea/swarm-legacy](https://github.com/miopea/swarm-legacy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-21 -->
