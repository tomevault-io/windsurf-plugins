---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

MCP server that exposes LTSpice circuit simulation to LLMs via the Model Context Protocol. LTspice is the primary simulator; ngspice/qspice/xyce are supported but secondary. Built on the low-level `mcp.server.lowlevel.Server` API (not FastMCP) with spicelib as the simulation backend.

## Commands

```bash
# Install dependencies (uses uv package manager)
uv sync

# Run the server (stdio transport)
uv run ltspice-mcp

# Type checking
uv run pyright

# Lint
uv run ruff check src/ tests/

# Lint with auto-fix
uv run ruff check --fix src/ tests/

# Format
uv run ruff format src/ tests/

# Run tests
uv run pytest tests/ -v

# Run a single test file
uv run pytest tests/test_pathutil.py -v

# Debug a single failure (disable parallelism for readable output)
uv run pytest -n0 tests/test_pathutil.py::TestName::test_case -v

# Run directly
python -m ltspice_mcp
```

No Makefile. CI: `.github/workflows/publish.yml` (test + publish to PyPI on version tags).

`pytest-xdist` is available, but the suite runs serially by default. Pass `-n auto` to parallelize locally when you do not need deterministic output order or debugger attachment.

`docs/TESTING.md` is the testing-practice doc: the absence-class blind spot path-walking stress tests cannot see (a missing or unusable-for-a-class capability), and the mechanisms that catch it — inverse-op closure (`test_dispatch.py::TestOpInverseClosure`), the archetype build battery (`test_circuit_asc.py::TestArchetypeBuildCoverage`), task-down coverage, and blind-artifact judging. Read it before adding a tool or an op.

## Comments, docstrings, and commit messages

These are read by people outside this repo's internal process — keep internal jargon out of them. Do **not** use, in code comments, docstrings, or commit messages: severity codes (P0–P3), internal codenames for stress-test findings (e.g. J-KILL, J-MAXPAR), backlog item numbers (`open_followups` item N), or stress-test version numbers (v9/v10). Describe the actual behavior, condition, or bug in plain technical terms instead. Internal planning docs under `.claude/plans/` and the backlog may use that shorthand; shipped code and git history may not.

## Architecture

All source lives under `src/ltspice_mcp/`.

### Layered Design

```
MCP Protocol Layer    server.py — lifespan, dispatch, request routing
                      resources.py — MCP resources & URI templates
Tool Layer            tools/*.py — tool definitions + handlers
Core Logic Layer      lib/*.py — see below
Config/State          config.py, state.py, errors.py
```

Key `lib/` modules:
- `services.py` — application-level service layer shared by tools and resources. Owns job resolution, cached result loading, and reusable extraction logic. Sits between MCP adapters and pure parsers. **Unified result read-model:** `runs_of(job)` projects either a `SimulationJob` or a `BatchJob` into a uniform `list[RunRef]` (a single run = batch-of-one); `resolve_run(job_id, run_index)` + `resolve_raw_file`/`resolve_log_file` address any run through it (gated on `completed`). This is the one place that knows the two physical result layouts (one multi-step raw vs N single-point raws), so extraction stays job-agnostic. `query_value`/`bode_metrics` accept `job_id`+`run_index` to analyze a sweep/MC run like a standalone raw (job-run raws bypass `safe_path` — trusted server artifacts, like `batch_results`).
- **SPICE lexer/validator** — `spice_lex.py` (foundation netlist lexer → `list[SpiceCard]` tokens), `spice_lex_ops.py` (cross-card transform passes), `spice_lex_views.py` (typed views over cards), `spice_validator.py` (pre-flight directive + arity validation). The `.cir`/`.net` read / list / value-edit paths and `validate_netlist` arity checks run on this pipeline, not spicelib's `SpiceEditor` (which is still used for some other `.cir`/`.net` editor ops).
- **Job subsystem** — `job_types.py` (domain dataclasses `SimulationJob`/`BatchJob`/`RunRef`/configs, re-exported from `state` to break import cycles), `job_registry.py` (in-memory registry + disk persistence + interrupted-job recovery + `preload_recent`), `job_store.py` (per-circuit JSON sidecar at `{circuit_dir}/.ltspice-mcp/jobs/{job_id}.json`), `job_lifecycle.py` (declarative status state machine, `transition()`).
- `sim_runner.py`, `sweep_runner.py`, `montecarlo_runner.py`, `runner_base.py` — spicelib runner wrappers (`runner_base` = shared scaffolding); `montecarlo.py` is the pure perturbation engine behind `montecarlo_runner`
- `runner_manager.py` — centralized runner lifecycle (see Key Patterns)
- `simulator.py` — simulator detection, WSL/Wine selection
- `ltspice_wsl.py`, `wsl.py` — WSL path conversion and interop
- `ac_analysis.py`, `signal_analysis.py` — pure-function analysis primitives for frequency-domain (.AC) and transient (.tran) `.raw` data; back the structured analysis tools (`bode_metrics`, `signal_stats`, etc.)
- `raw_parser.py`, `log_parser.py` — simulation `.raw` / `.log` result parsing
- `library_manager.py`, `library_parser.py`, `encoding.py` — component library handling + library/netlist encoding detection

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Cognitohazard/ltspice-mcp](https://github.com/Cognitohazard/ltspice-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
