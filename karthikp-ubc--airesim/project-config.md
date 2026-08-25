---
trigger: always_on
description: This file gives Claude Code the context needed to work effectively in this repo.
---

# CLAUDE.md — AIReSim

This file gives Claude Code the context needed to work effectively in this repo.

---

## Project overview

AIReSim is a discrete-event simulator (DES) built on [SimPy](https://simpy.readthedocs.io/)
for modeling reliability, failure recovery, scheduling, and repair in large-scale AI training
clusters.  The core concept: a job needs `job_size` servers running simultaneously; failures
interrupt the job and trigger a repair pipeline; the simulator measures end-to-end training
time and cluster efficiency (ETR — Effective Training Ratio).

---

## Setup

```bash
pip install -e ".[dev]"    # simpy + matplotlib + pytest + ruff
```

Python 3.10+ required.

---

## Essential commands

```bash
# Lint (must pass before any commit)
ruff check airesim/ tests/
ruff check airesim/ tests/ --fix   # auto-fix safe issues

# Tests (must pass before any commit)
pytest tests/ -v

# Both at once (mirrors CI)
ruff check airesim/ tests/ && pytest tests/ -v

# Run the built-in demo
python -m airesim.run

# Run a sweep script
python -m airesim.run examples/paper_table1_sweep.py

# CLI one-way sweep
python -m airesim.run --sweep recovery_time --values 10,20,30 --replications 30

# Adaptive replication from a YAML params file
python -m airesim.run --params config.yaml --adaptive
```

---

## Repository layout

```
airesim/                    # source package
├── params.py               # Params dataclass — every simulation knob
├── server.py               # Server entity: ServerState enum + state machine
├── coordinator.py          # Failure engine: aggregated exponential / Weibull / lognormal sampling
├── scheduler.py            # Host selection and warm-standby management
├── repairs.py              # RepairShop: auto → manual two-stage pipeline (SimPy processes)
├── pool.py                 # PoolManager: working pool / spare pool bookkeeping
├── scheduling_policies.py  # HostSelectionPolicy ABC + Default, FewestFailuresFirst,
│                           #   HighestScoreFirst, PackedByRackFirst
├── policies.py             # RepairEscalationPolicy + ServerRemovalPolicy ABCs;
│                           #   NeverRemove, ThresholdRemoval, ScoredRemoval,
│                           #   CompositeRemovalPolicy; re-exports scheduling_policies
├── topology.py             # Optional rack_id assignment (Params.enable_topology)
├── simulator.py            # Top-level DES orchestrator
├── stats.py                # StatsCollector (per-run), AggregateStats (multi-rep)
├── sweep.py                # OneWaySweep / TwoWaySweep parameter sweep drivers
├── adaptive.py             # AdaptiveRunner — auto-determines sufficient replications
├── plotting.py             # Matplotlib chart helpers (optional dep)
├── run.py                  # CLI entry point
└── __init__.py

tests/                      # 97 tests across 6 modules, all passing
├── test_airesim.py         # Core params, server, coordinator, pool, scheduler, sweeps (23)
├── test_edge_cases.py      # Race-condition / bug-regression tests (5)
├── test_scored_removal.py  # ScoredRemoval unit + integration (24)
├── test_scheduling_policies.py   # DefaultHostSelection, HighestScoreFirst ordering, reset (14)
├── test_diagnosis_probability.py # diagnosis_probability / diagnosis_uncertainty (18)
└── test_topology.py        # rack assignment, PackedByRackFirst, enable_topology flag (13)

docs/
├── ARCHITECTURE.md         # Module-by-module reference + design decisions
├── TUTORIAL.md             # Step-by-step user guide
└── *.md                    # Simulation reports (ETR, retirement, scheduling, heatmap, …)

examples/                   # Standalone sweep scripts; each produces a *_figures/ directory
config.yaml                 # Ready-to-use params file (paper defaults + adaptive settings)
```

---

## Code conventions

These are enforced by `ruff` and checked in CI.  Violating them will break the lint step.

| Rule | Detail |
|------|--------|
| **Style** | PEP 8; `ruff` rules E, W, F, I |
| **Line length** | ≤ 100 characters (`ruff` E501) |
| **Import order** | `ruff` I001 — stdlib → third-party → first-party; each group alphabetical |
| **Unused imports** | Not allowed (`ruff` F401); tests exempt from unused-variable F841 |
| **Type annotations** | Required on all public functions and methods |
| **Docstrings** | Required on all public classes, methods, functions |

Do **not** add new core dependencies to `airesim/` without discussion.  Optional deps
(e.g. matplotlib) go in `[project.optional-dependencies]` in `pyproject.toml`.

---

## Architecture essentials

### Key invariants

- **No circular imports.** Dependency order: `params` → `server` → `pool` →
  `scheduling_policies` → `policies` → `repairs/scheduler/coordinator` → `simulator` →
  `stats/sweep/adaptive`.  `plotting.py` only imports via `TYPE_CHECKING`.
  `topology.py` sits alongside `pool.py` — it only imports `server.py` and is
  itself imported by `simulator.py`.
- **`Params` is the single source of truth.** All simulation knobs live in the `Params`
  dataclass.  Use `params.with_overrides(**kwargs)` to create isolated copies for sweeps.
- **Pluggable policies via dependency injection.** `HostSelectionPolicy`,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [karthikp-ubc/airesim](https://github.com/karthikp-ubc/airesim) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-22 -->
