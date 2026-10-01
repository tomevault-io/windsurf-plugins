---
trigger: always_on
description: Read this before touching `winning.ratings`. Most of it exists because a
---

# winning — instructions for coding agents

Read this before touching `winning.ratings`. Most of it exists because a
session spent hours reinventing things the package already does.

## Pick the right filter FIRST

There are two ratings filters. Choosing the wrong one cost a machine reboot.

| | `AbilityTracker` (`ratings/tracker.py`) | `rate_history` / `walk_forward` (`ratings/history.py`) |
|---|---|---|
| state | one (mean, var) per entity, **no joint covariance** | dense N×N covariance over *every* entity |
| cost per contest | O(entrants) | O(N²) — plus eigendecompositions of the full N×N |
| use when | many entities, small fields (leagues, pairwise sports, ladders) | small closed fields where cross-covariances matter |
| drift | mean pegged, `v += drift·dt^drift_exp` (random walk, no level) | OU toward `prior_mean` with `timescale` — one rate for both reversion and uncertainty growth |

**Default to `AbilityTracker` for anything with hundreds of entities.**
A few hundred entities on the dense filter meant five full-matrix `eigh`
calls per contest and saturated every core; the tracker never builds that
matrix.

**Name trap:** `from winning.ratings import walk_forward` is the *dense*
`history.walk_forward`. The tracker's is only at
`winning.ratings.tracker.walk_forward`. Same return shape, plus `tracker`
and `evidence`.

## The caller owns the results → performance transform

`winning` is domain-agnostic. It never sees points, goals or seconds.

- Tracker: pass `scores` = performances already on the ability unit,
  **min-wins** (lower is better). `observe(..., scores=)` negates internally.
- Dense filter: pass `margins` (the gap to the winner in whatever unit
  the caller measures, lower is better) and `lengths_scale`, or `scores`.
  `lengths_scale` is the constant that converts those margins into
  performance units; the name is historical and carries no assumption
  about what the margin is.

Calibrate this before anything else, or prices and results land on two
different ability scales and the filter cannot be coherent. Recipe for
pairwise contests with a margin: find an unbiased forecast of the margin
(a published line, or a regression), take the residual sd σ, then

    lengths_scale = sqrt(2·beta2) / σ

A guessed value a third too small silently down-weights results against
prices and produces under-dispersed predictions that look like a modelling
problem. Check it with the regression in the diagnostics section.

## Prices are inverted under the outcome model — on purpose

Market prices go through `abilities_from_race` with race noise only. A
filter that has seen one price with noise `tau2` still has belief variance,
so its prediction sits closer to evens than the price it was given. That
shrinkage is correct Bayesian behaviour and vanishes as the belief sharpens.
It is **not** a bug; `tests/test_market_update.py::
test_update_race_market_leg_inverts_under_the_outcome_model` guards the
semantics. Do not "fix" it by inverting through the predictive covariance.

## Tuning: use the library's verdicts, not your eyes

Any sweep goes through `winning.ratings.tuning`:

    from winning.ratings.tuning import select, select_grid, require_live
    best, report = select({0.06: .0200, 0.13: .0125, 0.20: .0085}, "lengths_scale")
    print(report)   # -> lengths_scale=0.2  AT GRID EDGE: extend the grid

`select` reports INERT (a tie-break, not tuning), AT GRID EDGE (extend it)
and SATURATED (benign edge). `require_live` raises on an inert parameter.
`tune_block_rho` shows the pattern. A value chosen at a grid edge has not
been tuned; say so.

- **Do not tune a drift/`timescale` on a window with no season gap.** The
  parameter is unidentifiable there and Nelder-Mead will run it to
  decades.
- Evidence (`tune_history`, `tracker.evidence`) and held-out log-loss can
  disagree. Evidence on one season preferred "trust the price, ignore the
  results"; predictive log-loss wanted the opposite. Report which objective
  you used.
- Always validate on data the tuning never saw. A model/market gap can
  drift by season, and in-sample gains can halve out of sample.

## Run the tests the way CI runs them

    python -m pytest -q                 # bare: pytest.ini's testpaths, 750+ tests
    # NOT `pytest tests/` -- that is 460 of them; tests_arena, tests_research
    # and tests_experiments are collected only by the bare command.

CI installs `pip install -e ".[test]"`: pytest, pandas, matplotlib. It does
NOT have jax, fastrace, trueskill or sklearn, and this machine has all
four. A run that passes here with them present proves nothing about CI.
Before claiming green, run once with the optional backends blocked:

    # sitecustomize.py on PYTHONPATH: a PathFinder on sys.meta_path that
    # raises ImportError for jax, fastrace, trueskill, sklearn
    PYTHONPATH=<dir with that file> python -m pytest -q

Match a missing module on its MESSAGE ("No module named X"), never the
exception class -- a blocker raises ImportError where a real absence
raises ModuleNotFoundError, and a regex on the class name passes locally
and fails in CI. Three CI rounds were lost to exactly these in 2026-09.

One job at a time. The suite takes ~15 min alone and 50 min when the

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [microprediction/winning](https://github.com/microprediction/winning) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
