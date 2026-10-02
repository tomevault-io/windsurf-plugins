---
trigger: always_on
description: > **Breaking changes**: see `CHANGELOG.md` (when it lands) for migration notes.
---

# AGENTS.md

> **Breaking changes**: see `CHANGELOG.md` (when it lands) for migration notes.
> Until then, the invariants in `docs/architecture.md` §8 are the contract.

## Project overview

fintel is a benchmark evaluation platform for financial agents. A **strategy
package** is the benchmark (universe, schedule, data, mission, output contract,
signal, KPI). An **agent** is whatever you bring (in-process LLM desk, subprocess
CLI, scripted baseline). The platform runs the simulation under point-in-time
controls and scores the decisions post-run.

Read [`docs/architecture.md`](docs/architecture.md) first — especially §1
(the boundary and the financial-vs-terminal eval distinction) and §8
(invariants & layer map). The layer map is enforced by
`tests/test_architecture.py`; do not add an import that crosses it.

## Quick start

```bash
# Dev install
uv sync --all-extras

# Smoke run (2 names × 1 date)
fintel simulation packages/systematic_stockrate_djia_weekly \
  --agent optimized \
  --job-id smoke \
  --universe AAPL,MSFT \
  --dates 2026-04-24 \
  --shared-concurrency 4

# Score it
fintel report smoke
# same-period sidecar (leaves report.json in place):
# fintel report <longer-job> --start 2026-01-02

# Tests
uv run pytest tests/
```

Secrets: `mkdir -p .env && cp .env.example .env/keys.env`, then fill keys.
`.env/` is gitignored. Never commit keys.

## Repository structure

```
fintel/
├── fintel/                # the platform
│   ├── models/            # L0 — pure schemas, no I/O
│   ├── utils/             # L1 — generic helpers
│   ├── pit/               # L2 — the platform guarantee: clamp + audit
│   ├── market/            # L3 — catalog, universes, schedules, data sources, realized prices, probe, prefetch
│   ├── environment/       # L4 — session, access, mcp, evidence, nerve, echo, staging, cell (Cutoff owner)
│   ├── agents/            # L5 — adapters + agent-side context (barred from market/ and pit/)
│   ├── strategy/          # L6 — package load, lock, preflight
│   ├── simulate/          # L7 — job, run, trial, cell, queue, store, artifacts, backfill
│   ├── evaluate/          # L8 — read-only analytics: signals, kpi, holdings, behaviour, variance, report
│   └── cli/               # L9 — parse flags, build config, call simulate/evaluate, present, watch
├── packages/              # strategy packages (external, loaded not imported by platform)
├── docs/                  # architecture + guides
├── runs/                  # job output (gitignored; weekly demo + geopol runs negated in)
├── cache/                 # central data cache (gitignored)
└── tests/                 # pytest suite, flat
```

Imports flow one way only (L0 → L9). Enforced by `tests/test_architecture.py`.

## Key concepts

### Strategy package

An external directory the platform loads, validates, and freezes:

```
packages/my_strategy/
  strategy.toml          # manifest: universe, schedule, data bindings, scoring
  mission.md             # agent character / rubric
  alpha_view.md          # optional — standing thesis (composed into every prompt)
  alpha_views/           # optional — dated notes YYYY-MM-DD.md (PIT)
  output_schema.json     # View contract (symbol + score in [-1,1] minimum)
  strategy.lock          # written by load_and_prepare
  my_sources.py          # optional — bring-your-own DataSource
  scoring.py             # optional — custom signal / KPI callables
```

The platform never edits it. `load` parses; `preflight` reports every reason
it can't run; `build_lock` freezes identity. See
[`docs/add_strategy_package_guide.md`](docs/add_strategy_package_guide.md).

### Execution hierarchy

```
Job → Run → Trial → Cell
```

- **Job** — one `fintel simulation` invocation: package × agent × market × K
- **Run** — one of K repeats (the stochasticity unit)
- **Trial** — one decision date
- **Cell** — one agent invocation (per-symbol for `single_name`, whole universe for `portfolio`)

Cell is the load-bearing orchestration level: it owns the `Cutoff` (only
`environment/cell.py` may construct one), and it's where retry lives.

### Agents

`agents/` holds one adapter per agent behind an `AgentFactory` name map.
Contract: `decide(environment) -> AgentResponse` — a returned value, not a
callback. Two hosts:

- **in-process** (`llm`, `scripted`, `constant`, `optimized`) — consume
  `DataAccess` / tool objects directly.
- **subprocess CLI + MCP** (`openclaw`, `claude_code`) — fintel MCP server
  rebuilds `Environment` from the session dir.

Every adapter declares `pit_enforcement`: `access` (in-process, already
clamped) or `cli_deny` (subprocess, deny list + MCP isolation per cell).

### Evaluation

Read-only over finished jobs, never imports `simulate/`. The strategy owns
the signal (`signal_fn(views) → {symbol: float}`) and KPI
(`kpi_fn(signal_by_date, prices, horizons, params) → dict`); the platform owns
ensemble (cell-mean across K), holdings, returns, behaviour (L1), variance (L2),
report rendering.

## Development setup

```bash
git clone https://github.com/felixdaga/fintel.git
cd fintel
uv sync --all-extras
```

Python ≥ 3.11. Ruff for format + lint, pytest for tests.

```bash
uv run ruff format .
uv run ruff check --fix .
uv run pytest tests/
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [felixdaga/fintel](https://github.com/felixdaga/fintel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
