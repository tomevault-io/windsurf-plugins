---
trigger: always_on
description: Vendor-neutral rules for AI coding agents working on EMHASS source.
---


<!-- Last verified against upstream/master @ 6537c47, 2026-04-30 -->

Rules for AI coders (Claude Code, Cursor, Aider, Copilot, Codex) on EMHASS source. Complements `docs/develop.md` (human canon); no duplication. Where `develop.md` covers topic, this links + adds AI-specific constraints.

*Humans driving agent: see [`docs/develop_ai_coders.md`](docs/develop_ai_coders.md) for contributor companion.*

## Repository layout

- `src/emhass/` — `optimization.py`, `forecast.py`, `retrieve_hass.py`, `web_server.py`, `command_line.py`, `utils.py`.
- `tests/` — pytest suite.
- `docs/` — Sphinx source. Start: `develop.md`; examples: `study_cases/`.
- `data/` — `config_defaults.json`, `associations.csv`.
- `src/emhass/static/` — web UI assets incl. `param_definitions.json`.

## Section 1 — Canonical commands

Full setup + workflow in `docs/develop.md` (Method 1: `uv` venv, Method 2: DevContainer, Method 3: Docker). Read first.

Quick-recall:

| Action | Command |
|---|---|
| Run tests | `pytest tests/` |
| Sync dev deps | `uv sync --extra test` |
| Build docs | `sphinx-build -b html docs docs/_build` (configured in `docs/conf.py`) |
| Lint | `uvx ruff check .` (enforced via `.github/workflows/code-quality.yml` on every PR) |

Tech stack (verify versions in `pyproject.toml` before assuming API):

| Component | Version source |
|---|---|
| Python | `pyproject.toml` `requires-python` |
| Optimisation | CVXPY (pin in `pyproject.toml`) |
| Web | Quart + Uvicorn |
| Tests | pytest |

## Section 2 — Pipeline map

Three optimisation modes via `command_line.py`. Shared: input-prep + publish. Body differs by mode. Core: one method on `Optimization`.

| Phase | Symbol |
|---|---|
| Input preparation | `command_line.py::set_input_data_dict` |
| Mode entry, perfect forecast | `command_line.py::perfect_forecast_optim` |
| Mode entry, day-ahead | `command_line.py::dayahead_forecast_optim` |
| Mode entry, rolling MPC | `command_line.py::naive_mpc_optim` |
| Optimisation core (LP build and CVXPY solve) | `optimization.py::Optimization.perform_optimization` |
| Publish | `command_line.py::publish_data` |

Stage instrumentation: `stage_timer(stage_times, "<label>", logger)`. Five active labels:

- `"pv_forecast"`
- `"load_forecast"`
- `"price_prep"`
- `"optim_solve"`
- `"publish"`

Grep `'stage_timer.*"<label>"'` for live call site. Labels stable; line numbers not.

## Section 3 — Don't-touch rules

Five invariants. Easy to break, hard to detect in CI.

1. **`action_logs.txt` line format.** `web_server.py` parses lines by splitting on first whitespace, comparing leading token to `"ERROR"`. Format change (prefix, JSON, structured log) silently breaks UI error reporting.

2. **Logger handler accumulation in `utils.get_logger`.** Attaches handler unconditionally every call. Duplicate calls → duplicate log lines → masked failures. Avoid. Guard changes need maintainer coordination (CLI + web both call this).

3. **Two parallel logging subsystems.** CLI: `utils.get_logger`. Web: `app.logger`. Touch both or neither. Partial migrations break downstream log consumers.

4. **`param_definitions.json` is a structured surface.** Additive only. Rename/remove/type-change breaks config UI + external tooling. New entries OK; mutations need migration plan + maintainer review.

5. **Optimisation-result DataFrame columns and units.** `opt_res_*` → `opt_res_latest.csv`, HA sensors, external bridges. Column renames + unit changes = breaking. Additive columns OK. Current `opt_res_latest.csv` columns = de-facto schema.

## Section 4 — Maintainer scope corridors

From public maintainer statements. Cite source if questioned.

- **Threat model** (#808): security scope = code injection, not auth bypass/data leakage. In-memory-read endpoints OK; filesystem/DB/shell endpoints need maintainer sign-off.
- **EMHASS scope** (#789): MILP optimiser. Vehicle APIs, OCPP, EVCC, charger modulation → integration layer, not core.
- **Glue layer agnostic.** Node-RED, MQTT, HA, generic automations = equivalent. No HA-specific code in core.
- **Zero-config default must keep working.** Add-on starts + produces sensible optimisation on defaults after every change.

## Section 5 — Limits and gotchas (read this if you are an AI coder or working with one)

AI finds code + candidates. Domain experts decide bug vs design. 2026-04-26 audit: 8 findings, 4 PR-able, 4 issue-first. Skip human-in-loop → ~50% noise.

**Issue first, not PR, when:**

- Behaviour changes visibly (output values, log format, error messages).
- Magic constant/sentinel might be intentional (`=0` = no constraint? negative = disabled?).
- Condition looks wrong but may encode domain convention (AC/DC power; charge/discharge sign).
- Change touches `optimization.py`, `retrieve_hass.py`, or `forecast.py` beyond ~3 lines.

**Verify before done:**

- Sign conventions (`P_grid > 0` = import or export? Check, don't assume).
- Units (HA scales SOC by 100; CSV uses 0..1).
- Test reproducer present for any behaviour-change PR.
- Smoke-test (`docker compose up` + browser config page) if schema or `web_server.py` changed.

**No refactor without issue:**

- Restructuring `optimization.py` (3000+ lines) without RFC issue → rejected.
- Renaming public API params breaks downstream; needs migration path.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [davidusb-geek/emhass](https://github.com/davidusb-geek/emhass) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
