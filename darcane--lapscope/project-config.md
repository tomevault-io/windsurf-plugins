---
trigger: always_on
description: Working knowledge for AI agents (and humans) touching this repo: workflow rules
---

# AGENTS.md — LapScope

Working knowledge for AI agents (and humans) touching this repo: workflow rules
and the hard-won behavioral facts that must not be re-derived or regressed.

## Documentation map — read in this order

1. **This file** — dev workflow, FH6 packet facts, the event-detection model,
   test matrix. The *why* behind the code.
2. **[ARCHITECTURE.md](ARCHITECTURE.md)** — what lives where: file
   responsibilities, data flow, DB schema, API surface, WebSocket contract,
   concurrency rules, simulator flags, and the cross-file invariants that must
   stay in sync. Consult it before touching anything structural.
3. **[README.md](README.md)** — user-facing: setup, in-game settings, quick
   troubleshooting. Kept deliberately basic — deep material goes on the wiki.
4. **Wiki** (source: [docs/wiki/](docs/wiki/)) — user-facing deep dives:
   Troubleshooting, Capturing-an-Unrecognized-Event, FH6-Data-Out-Packet,
   Event-Detection (+ `_Sidebar`/`_Footer`). Mirrored to the GitHub wiki by
   [.github/workflows/wiki.yml](.github/workflows/wiki.yml) on every merge to
   `main`; `docs/wiki/` is the **source of truth — never edit the wiki on
   GitHub**, the next sync overwrites it. Pages link each other wiki-style
   (`[text](Page-Name)`, no `.md` — resolves only on the published wiki) and
   link repo files/images by absolute URL
   (`https://github.com/darcane/LapScope/blob/main/…`,
   `https://raw.githubusercontent.com/darcane/LapScope/main/…`).
5. **[GitHub Issues](https://github.com/darcane/LapScope/issues)** — the open
   backlog: bugs, feature ideas, and in-flight investigations. (The old
   `TODO.md` is retired; everything it tracked lives in the issue tracker now.)

Keep all five current: a detection change usually touches this file's model
section **and** the wiki's Event-Detection page; a new endpoint/table belongs
in ARCHITECTURE.md; a new user-visible feature in the README; a change to
packet understanding, troubleshooting steps, or the capture workflow updates
the matching `docs/wiki/` page **in the same PR**; an issue gets closed when
its work lands, and new bugs/ideas discovered along the way get filed as
issues — never appended to a file.

## What this is

A single-container FastAPI app that receives Forza Horizon 6 "Data Out" UDP
telemetry, shows a live dashboard, and records sessions/laps into SQLite for lap
analysis. Vanilla-JS frontend, no build step. The recorder's decisions are
covered by a fast headless `pytest` harness (drives the simulator's scenarios
straight through `SessionTracker`, no game or container — see
[tests/](tests/)); running the simulator against the live container is still the
way to verify the frontend and the real UDP path.

```
FH6 ──UDP 9999──▶ listener.py ─▶ packet.py parse ─┬─▶ hub.py ─▶ /ws/live ─▶ dashboard.js
                                                  └─▶ laps.py (SessionTracker) ─▶ store.py (SQLite)
                                                        REST /api (routes.py) ─▶ analysis.js
```

## Dev workflow (important, non-obvious)

- **After each iteration of changes, create a local git commit.** **Never push
  to a remote** — the owner handles pushing themselves.
- **Work item → branch → PR.** Pick an item from the
  [issue tracker](https://github.com/darcane/LapScope/issues), cut a `feat/…`
  or `fix/…` branch off `main`, commit there, and land it via a pull request
  with CI green — not by committing straight to `main`. `main` is the
  protected, release branch. The owner pushes and opens the PR.
- **Static files are baked into the image.** Any change under `app/` requires
  `docker compose build` + restart. There is no bind mount for code.
- **Only `desktop/ui.py` may import tkinter.** The rest of the launcher
  (`state.py`, `logs.py`, `server.py`, `paths.py`) has to stay importable on a
  headless Linux box, because that is where CI runs and it is the only reason
  any of that logic is testable. Put decisions in those modules and keep
  `ui.py` to painting and event wiring. `tests/test_desktop_no_tkinter.py`
  fails if this slips.
- **The exe is windowed** (`console=False`), so `sys.stdout` and `sys.stderr`
  are `None` in the frozen build: nothing on the exe's import path may
  `print()`. Anything the user needs to see goes through `logging` — it reaches
  both the window's log pane and `DATA_DIR/logs/lapscope.log`. CI never builds
  the exe; the release workflow's `LS_DESKTOP_SELFTEST` step is what catches
  this class of break, so run it locally if you touch `run_desktop.py`,
  `LapScope.spec`, or `desktop/`.
- The Claude Code preview config (`.claude/launch.json`) runs `docker compose up`
  and owns the process — stop the preview server AND `docker compose down` before
  rebuilding, then start the preview again.
- **Run the tests before opening a PR** — CI runs the same on every PR. From the
  repo root: `pytest -q` and `ruff check .` (install the tooling once with
  `pip install -r requirements-dev.txt`). The recorder scenarios in
  `tests/test_scenarios.py` are the matrix at the bottom of this file, driven
  headlessly through `SessionTracker` in ~2 s by a fake-socket harness
  (`tests/harness.py`) — no container, no real-time wait.
- Verify with the simulator (no game needed), from the repo root:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [darcane/LapScope](https://github.com/darcane/LapScope) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-31 -->
