---
trigger: always_on
description: Agent bootstrap and operational notes, for any coding agent (Claude Code,
---

# AGENTS.md — Jaxonomy

Agent bootstrap and operational notes, for any coding agent (Claude Code,
Codex, Gemini, Cursor, …) and human contributors. This is the **canonical,
tool-neutral entry file**; `CLAUDE.md`, `GEMINI.md`,
`.github/copilot-instructions.md`, and `CONVENTIONS.md` are symlinks to it, and
`.cursor/rules/` points here (see "Entry points" at the end).

Two doors, depending on what you're here to do:

- **Modifying / adding code in Jaxonomy** → follow the read order below,
  starting at `AGENTS/README.md`. Full project orientation lives in `AGENTS/`;
  come back here for the things that specifically save agent time.
- **Using Jaxonomy's public API** in your own code (authoring a tutorial,
  building a demo, writing a downstream library) → read `SKILL.md`, the
  consumer operating manual, instead.

## Read first

1. `AGENTS/README.md` — navigation + which AGENTS file to use when.
2. `AGENTS/CONTEXT.md` — what Jaxonomy is, design philosophy, key abstractions.
3. `AGENTS/PATTERNS.md` — coding conventions (`npa` vs `jnp`, `LeafSystem`
   callback signatures, NamedTuple state, test patterns, naming).
4. `AGENTS/DECISIONS.md` — ADRs; check before re-litigating a settled choice.
5. `AGENTS/RULES.md` — operating principles, shippable-surface rule, claims/gaps
   discipline, self-improvement loop.

For a pure usage session (author a tutorial, build a demo, exercise the public
API), `SKILL.md` is the better starting point.

## Operating discipline (pointers, not a second copy)

The substance lives in two files; this bootstrap defers to them rather than
restating them:

- **`AGENTS/RULES.md`** — the four operating principles (think before coding;
  simplest implementation that fits; surgical changes only; define success then
  loop), the shippable-surface rule + adversarial-review pass, claims/gaps
  discipline, and the self-improvement loop. Read it once.
- **`AGENTS/README.md`** — session protocol, branching/commits, scope
  discipline, and the autonomy/escalation list.

Two reminders that bite most often in an agent session: changes to a
shippable surface (`README.md`, `docs/**`, `examples/**`, `benchmarks/**`, root
`*.ipynb`, `CLAIMS.md`, `KNOWN_GAPS.md`) must be real and evidence-backed —
removed beats fake; and an unrelated bug found mid-task is surfaced to the
maintainer (commit message / PR), not fixed as a drive-by on your branch.

## Where things actually live (non-obvious)

| Looking for                                  | File                                                      |
|----------------------------------------------|-----------------------------------------------------------|
| `findop`, `frequency_response`, `bode_data`, `nyquist_data`, `pole_zero_map`, `step_response`, `impulse_response`, `estimate_frequency_response` | `jaxonomy/library/linearization_workflow.py` (**not** `jaxonomy/optimization/` despite the name) |
| `linearize`, `LinearizedSystem`, `LTISystem`, `TransferFunction`, `PID` | `jaxonomy/library/linear_system.py`                       |
| Standard-library blocks — split by category (was `primitives.py` until the refactor) | `library/sources.py` (sources + stochastic), `library/math_ops.py`, `library/logic.py`, `library/routing.py` (mux/demux/buses), `library/dynamics.py` (integrators + discrete state + filters + PID), `library/nonlinearities.py` (saturate / dead zone / rate limiter / quantizer), `library/tables.py` (lookup family). `library/primitives.py` is a re-export hub for back-compat — `from .primitives import X` keeps working. |
| Container blocks (`EnabledSubsystem`, `TriggeredSubsystem`, `ForEach`) | `jaxonomy/framework/containers.py`                        |
| Unit annotations (`BusUnit`)                 | `jaxonomy/framework/units.py`                             |
| Variants                                     | `jaxonomy/framework/variants.py`                          |
| Lookup-table fitting (`fit_lookup_table_*`)  | `jaxonomy/library/lookup_table_fitting.py`                |
| UQ workflow (Monte Carlo, Sobol, LHS, qMC)   | `jaxonomy/uq/`                                            |
| Diagnostics (dead-store, empty-inputs)       | `jaxonomy/diagnostics.py`                                 |
| Parameter tuning helpers                     | `jaxonomy/optimization/parameter_tuning.py`               |
| Lazy results + DuckDB backend                | `jaxonomy/simulation/lazy_results.py`                     |
| Event-time gradient infrastructure           | `jaxonomy/simulation/event_gradient.py`                   |
| Fast restart                                 | `jaxonomy/simulation/fast_restart.py`                     |
| Provenance manifest                          | `jaxonomy/simulation/provenance.py`                       |

## Testing

- Fast tier: `pytest -m "not slow"`. Use this for regression checks on
  routine work.
- Tests live in `test/` (singular), mirroring the source layout.
- **Baseline: fully green.** A full-suite run (2026-07-09, all 5109 tests,
  fast + slow tiers) had zero unexpected failures — the previously listed
  baseline failures (Kalman filters, state-machine dtype, random
  normal/gamma, fluid, `battery_cell`, `edge_detection_comparator`) all
  pass now. `test_predictor.py` Torch/TF tests *skip* when

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [machinavitalis/jaxonomy](https://github.com/machinavitalis/jaxonomy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-31 -->
