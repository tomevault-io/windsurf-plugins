---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

Shelf — a self-hosted home library catalog. FastAPI + SQLite (raw `sqlite3`, no ORM) backend; server-rendered Jinja2 + HTMX + Alpine.js (CSP build) + Tailwind frontend. Single Docker container, HTTPS with self-signed certs, AGPL-3.0.

Known traps live in `GOTCHAS.md` — check it before touching migrations, Alpine components, or anything it triggers on.

## Commands

Run all `make` targets from inside this directory (`cd shelf && make ...`, never `make -C shelf` — some targets use git commands that break).

```bash
make setup                 # one-time: dev deps + npm (tailwind) + Playwright Chromium
make test                  # unit/integration tests — quiet + parallel (excludes tests/e2e/)
make test-fast             # re-run only last run's failures (--lf, serial)
make test-verbose          # per-test roll-call, for humans
make test-e2e              # Playwright E2E — spins up its own server, no dev server needed
make test-contract         # live UPC Item DB contract check — run at release, never on a gate
python -m pytest tests/test_items.py::test_name -v           # single unit test
python -m pytest tests/e2e/test_scan.py -v -m e2e            # single E2E file
make css                   # rebuild committed Tailwind stylesheet + restamp SW_VERSION (required after template/JS changes; right on a monorepo branch, refused on a GitHub PR)
make check-csrf            # lint: raw fetch() calls must send X-CSRF-Token
make check-deleted         # lint: reads of items/item_copies must go through items_live/copies_live
make check-alpine          # lint: templates stay compatible with Alpine CSP build
make check-sw-version      # lint: SW_VERSION matches the precache digest (generated, never hand-edited)
make badges                # restamp README's test-count badges (generated — run after adding/deleting tests)
make check-badges          # lint: README test-count badges match what pytest collects
make check-tests           # lint: test conventions — app.main import isolation, CSP-safe waits, page guards, no networkidle wait after a click
make checks-fast           # instant offline lints (secrets, csrf, alpine, sw-version, tests, badges) — the inner-loop target
make checks                # everything, incl. network pip-audit + licenses — before a release
make dev / dev-down / dev-logs   # docker compose up/down/logs
uvicorn app.main:app --reload    # run without Docker
```

**Unit and E2E tests cannot run in a single pytest invocation** — always use the separate targets above.

### Agent-efficiency conventions

The dev loop is tuned for agent sessions, where command output is read into a context window
far more often than by a human. Keep it that way:

- **`make test` is quiet and parallel** (`-q --tb=short --no-header`, `-n auto --dist loadfile`).
  It prints failures, not a roll-call — 917 `PASSED` lines is roughly 15k wasted tokens per run.
  Reach for `make test-verbose` only when a human is reading, and `make test-fast` while
  iterating on a specific failure.
- **Run the slow things in the background** — `make test-e2e`, `docker compose build`, and
  `make release` all take minutes. Launch them with Bash `run_in_background: true` and do other
  work; the harness re-invokes on exit. Never launch in the background and then `sleep`-poll —
  the completion notification *is* the signal. **But "other work" cannot mean editing the tree
  those targets are reading.** `make test-e2e` serves the working copy through its own uvicorn,
  so a source edit landing mid-run makes the result unattributable — you can no longer tell a
  pre-existing failure from one you just caused, which is exactly what a baseline run exists to
  establish. Stash or wait; docs and other files nothing under test imports are fair game.
  **Not under a headless run.** A pipeline stage (`plan-pipeline.sh`, `post-run-pipeline.sh`
  — anything driven by `claude -p`) gets no completion notification: ending the turn ends the
  process. There, run probes and test commands in the **foreground** and wait for the output.
  Measured 2026-09-01: `/impl-plan` backgrounded a pytest probe, ended its turn "until the
  notice arrives", exited 0 after 11 min with no plan written.
- **`make checks-fast` in the loop, `make checks` before a release.** The full target runs
  `pip-audit` over the network and writes dated reports into `reports/`.
- **Prefer the `gh` CLI over the `github` MCP tools here.** The release procedure in
  `../CLAUDE.md` already uses `gh` end to end, and `gh ... --jq` lets you bound the output size,
  which the MCP tools do not.
- If you add a dependency that spams warnings, silence it **by message** in a
  `filterwarnings` stanza in `pytest.ini` (add the key if absent), never by blanket
  category — a muted category hides real deprecations.

## Architecture

**`docs/architecture.md` is the authoritative description** — what each middleware
is for, the data model and table set, the metadata and cover cascades and *why*
they are ordered that way, outbound pacing, the Photo Intake flow and its stated

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dgahagan/shelf](https://github.com/dgahagan/shelf) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
