---
trigger: always_on
description: Experiential builds immutable task evidence from agent traces, composes and fits frozen model routers,
---

# Agent guide — experiential

Experiential builds immutable task evidence from agent traces, composes and fits frozen model routers,
runs those routers on loopback, and executes bounded SFT from persisted datasets. All importable
code lives under `exp/`; benchmark data arrives as a dependency (see rule 6).

## Toolchain

Managed with `uv`; lint/format with `ruff`; type-check with `ty`.

```bash
uv sync --extra dev
uv run ruff check .
uv run ruff format --check .
uv run ty check
uv run pytest -q
```

## Repository checks

- Every new or rewritten hand-authored source, configuration, and documentation file with a
  covered suffix stays below 1,000 physical lines. The executable limit is 999 lines and counts
  comments and blank lines. Test modules named `*_test.py` are exempt so cohesive tests are not
  split solely to satisfy a line count. Generated lock files are excluded. Generated code belongs
  in an explicitly named `generated/` directory and is never edited by hand.
- Full-repository Ruff check, Ruff format check, and ty check are required on every change. No
  pre-existing lint or type failures are grandfathered.
- Production imports follow the approved dependency direction: common may not import runtime,
  simulation, optimize, or cli; runtime may not import simulation, optimize, or cli; simulation
  may not import optimize or cli; optimize may not import cli. Optimize owns application
  orchestration and may depend inward on common, runtime, and simulation. The AST gate rejects
  every current forbidden edge directly and proves that the package graph is acyclic.
- The root CLI command set is exact: `build`, `config`, `optimize`, and `run`;
  `exp/cli/app_test.py` and the release tests enforce the current command and distribution shape.

## CLI package ownership

- `exp/cli/app.py` owns root command composition only. Command implementations live in the
  `build/`, `config/`, `judge/`, `optimize/`, `run/`, and `gateway/` packages.
- `exp/cli/providers/` owns provider discovery, model selection, and catalog setup shared by
  commands. Command-specific orchestration stays with its command package. In particular,
  router-candidate collection belongs to `exp/cli/optimize/`.
- `exp/cli/shared/` owns reusable terminal, Typer, consent, picker, and progress primitives. It
  must not import command packages. `exp/cli/providers/` may import `shared/`, but it must not
  import command packages.
- Keep the `exp/cli/` root closed to new command implementation modules. Package-wide CLI tests
  may live in `exp/cli/tests/`; module tests stay beside the module they cover.

## Evidence, simulation, and routing lifecycle

- `exp/simulation/` owns trace ingestion, representative-task mining, typed simulation specs,
  current engines, orchestration, artifact construction, and comparisons. New modules for those
  responsibilities go inside `exp/simulation/`, never at the flat `exp/` root.
- `exp build PROJECT --traces TRACE_FILE --source SOURCE --root ROOT` is the only CLI path
  from local traces to immutable task evidence. It accepts 100 through 1000 normalized traces,
  writes manifest-bound fit and held-out tasks plus `proposals_pending` review state, builds both
  RAG indexes under a strict embedding-cost ceiling, and binds the grounded world model without a
  completion or judge call. Route each corpus through an explicit canonical source loader.
- New trace sources belong in `exp/simulation/ingest/`, normalize into the `Trace` and `TraceSpan`
  contracts in `exp/common/traces/`, support file ingestion, and register from
  `exp/simulation/ingest/__init__.py`.
- Python applications use `exp.compose_router` to complete review, plan-bound simulation,
  judgment, fitting, held-out verification, reporting, and runtime loading. Callers inject the
  approved review and setup suppliers, simulator factory, judge, runtime catalog, and finite
  simulation-dollar and judgment-call ceilings. Preserve its phase boundary: held-out evidence
  opens only after fit evidence, policy locking, and remaining-budget checks pass. Router fitting
  never runs or consumes world-model fidelity evaluation.
- `exp optimize router PROJECT --root ROOT` discovers the completed build, fit-only RAG, grounded
  world model, judge syllabus and provenance, and confirmed router candidates from the project.
  Human calibration is recommended but optional. The command freezes one shared provider ceiling
  before calls, simulates and judges missing fit evidence, locks the fit policy before held-out
  execution, and exactly replays completed work.
  World-model fidelity testing is a separately invoked common-evaluation mode with no authority
  over router fitting or runtime activation. Its reports contain measurements only and never carry
  an approval, denial, gate, threshold, or decision.
- `exp run` starts the initialized authenticated multi-alias gateway on loopback.
  `exp run PROJECT --root ROOT --port PORT [--ghost]` retains the single-project compatibility
  server. Both expose OpenAI Chat Completions, Responses, and Models routes. Public request and
  response types come
  from the official OpenAI SDK. Chat retries use the standard `Idempotency-Key`; Responses

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [experientiallabs/experiential](https://github.com/experientiallabs/experiential) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-20 -->
