---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**sim** is a unified CLI + HTTP runtime that lets LLM agents (and engineers) launch, drive, and observe CAD/CAE simulations across multiple solvers through one consistent interface. It is the "container runtime for simulations" — agents talk to `sim`, `sim` talks to solvers.

The runtime supports two execution modes:

- **One-shot** (`sim run script --solver=X`): subprocess execution, result stored as a numbered run, `sim logs` to browse.
- **Persistent session** (`sim serve` + `sim connect/exec/inspect/disconnect`): a long-lived HTTP server holds a live solver session; agents send code snippets and inspect state without restarting the solver.

The companion repo `sim-skills/` contains per-solver agent skills, reference docs, demo workflows, and integration tests that drive this runtime.

## Commands

```bash
# Install
uv pip install -e ".[dev]"          # core + pytest + ruff

# Tests
pytest -q                            # unit tests (no solver needed)
pytest tests/test_lint.py            # single test file
pytest -q -m integration             # integration tests (need solvers + sim serve)

# Lint
ruff check src/sim tests
ruff check --fix src/sim tests

# CLI
sim serve --host 0.0.0.0             # start HTTP server (default port 7600)
sim --host <ip> connect --solver fluent --mode solver --ui-mode gui
sim --host <ip> exec "solver.settings.mesh.check()"
sim --host <ip> inspect session.summary
sim --host <ip> screenshot -o shot.png
sim --host <ip> disconnect

sim run script.py --solver pybamm    # one-shot mode
sim logs                              # list runs
sim logs last --field voltage_V      # extract a parsed field
sim check fluent                      # solver availability
sim lint script.py                    # validate before running
```

Environment variables: `SIM_HOST`, `SIM_PORT` (CLI client, also `[server]` in config), `SIM_HOME` (global config + history dir, default `~/.sim/`), `SIM_DIR` (project dir, default `./.sim/`).

Config files (issue #5): `~/.sim/config.toml` (global) + `.sim/config.toml` (project). Resolution order `env > project > global > default`. With no config files present, behavior is unchanged from pre-config sim. Run `sim config path | show | init` to manage. See `docs/architecture/multi-session-and-config.md` for the full schema.

## Architecture

### CLI (`src/sim/cli.py`)
Click app with subcommands: `serve`, `check`, `lint`, `run`, `connect`, `exec`, `inspect`, `ps`, `disconnect`, `screenshot`, `logs`. The session-related commands (`connect`/`exec`/`inspect`/`ps`/`disconnect`/`screenshot`) all delegate to `sim.session.SessionClient`, an HTTP client that talks to a running `sim serve`. The non-session commands (`run`, `lint`, `check`, `logs`) work locally without a server.

### HTTP server (`src/sim/server.py`)
FastAPI app exposing:
- `POST /connect` — launch a solver, register a new session in `_sessions: dict[str, SessionState]` keyed by session_id
- `POST /exec` — `exec()` a Python snippet against the live `session`/`meshing`/`solver` namespace for the session selected by `X-Sim-Session` header (or the single live session if unambiguous); capture stdout/stderr/return value, append to that session's runs
- `GET /inspect/<name>` — query `session.summary`, `session.mode`, `last.result`, `workflow.summary` (session-scoped)
- `POST /run` — one-shot script execution (no session required)
- `GET /ps` — list of all live sessions + default_session (set only when exactly one live)
- `GET /screenshot` — base64 PNG of the server's desktop
- `POST /disconnect` — tear down the session selected by `X-Sim-Session` (or the sole live session)
- `POST /shutdown` — tear down all sessions, exit the server process

The server supports multiple concurrent sessions keyed by session_id. Each `SessionState` carries its own `threading.Lock` so exec/inspect against different sessions can run in parallel. A single solver name can only be live once (driver instances are module-level singletons).

**`sim serve --reload` drops all sessions on any source change under the watched tree.** uvicorn's reload watchdog observes file mtimes in `src/sim/**`; any edit (git pull, scp of a modified driver, even touching an unrelated module) restarts the worker, wiping `_sessions`. Child solver processes (Flotherm GUI, Fluent, etc.) survive the reload because they're spawned separately, but the session handles to them are gone — you have to `connect` again. Driver temp files (written to the solver's workspace, e.g. `flouser/_sim_*.xml`) live outside `src/` so they don't retrigger. Practical rules:

- Don't edit driver code mid-experiment; finish the run, then edit.
- For long autonomous experiments where you're editing driver code iteratively, launch **without** `--reload` and restart manually when you want the new code picked up.
- Reconnecting after a reload takes ~20s for GUI-mode Flotherm (the existing GUI process is re-adopted; no re-launch needed).

### Driver protocol (`src/sim/driver.py`)
`DriverProtocol` (a `runtime_checkable` `Protocol`):
- `name: str` — registered driver name

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [svd-ai-lab/sim-cli](https://github.com/svd-ai-lab/sim-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-26 -->
