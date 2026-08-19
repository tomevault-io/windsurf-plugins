---
trigger: always_on
description: Guidance for AI agents working in this repository. The documents under [docs/](docs/) are canonical for everything they
---

# AGENTS.md

Guidance for AI agents working in this repository. The documents under [docs/](docs/) are canonical for everything they
cover; this file adds only what is agent-specific and points at the rest.

## Project in one paragraph

`polars-stats` is a Polars expression plugin that exposes `scipy.stats`-style distributions natively inside Polars
expressions, with column-valued parameters. Rust + statrs handle the math, Python provides the user-facing distribution
classes.

## Where to look first

| You want to know | Read |
|---|---|
| Build / test / lint commands, the dependency stack, and the step-by-step for adding a distribution | [Contributing](./docs/contributing.md) |
| What the system is and how it is wired | [Architecture](./docs/explanation/architecture.md) |
| Why a decision was made, or what is still open | [Design notes](./docs/explanation/design.md) |
| Public method surface and base-class defaults | [polars_stats/distributions/_base.py](polars_stats/distributions/_base.py) |
| Canonical pattern, closed-form continuous | [_uniform.py](polars_stats/distributions/_uniform.py) / [uniform.rs](src/distributions/uniform.rs) |
| Canonical pattern, statrs-backed continuous | [_normal.py](polars_stats/distributions/_normal.py) / [normal.rs](src/distributions/normal.rs) |
| Canonical pattern, discrete | [_bernoulli.py](polars_stats/distributions/_bernoulli.py) / [bernoulli.rs](src/distributions/bernoulli.rs) |
| Canonical test layout | [tests/distributions/bernoulli/](tests/distributions/bernoulli/) |

## Conventions

Code, prose, test, and git conventions live in [Contributing > Conventions](./docs/contributing.md#conventions);
follow them, do not re-derive them here. On top of those:

* **Priorities, in order: correctness, ergonomics, maintainability, performance.** Performance last is deliberate:
  the polars engine does the heavy lifting on large frames, so a clear formula beats a fast one. Only reject a
  choice on performance grounds when it is clearly suboptimal (an `O(n)` draw per row, a per-draw rebuild), not to
  shave constants.
* Comments never narrate what the code does, and never reference tasks, fixes, or callers (that is PR-description
  material). Docstrings that pin a contract or invariant (null propagation, seeding, fast-path bit-equality) are the
  house style; everything else defaults to no comment.

## How to add a new distribution

The steps (Rust plugin surface, Python subclass shape, test files) are in
[Contributing > Adding a distribution](./docs/contributing.md#adding-a-distribution).

What matters specifically for an agent:

* **Work from the map, not the tree.** The architecture is already documented; read the contributing steps, the
  canonical pair for your family (closed-form `Uniform` vs statrs-backed `Normal`, plus `Bernoulli` for discrete),
  and the matching `tests/distributions/<canonical>/` rather than reverse-engineering the codebase. When you genuinely
  must explore further, fan the reads out in parallel. Resolve the choices the issue flags (a validator arity, a scipy
  reparam, an out-of-regime convention) up front from the issue file, not mid-implementation. In a repo this
  invariant-dense, understanding the fast-path / null / parity contracts beats debugging the property suite that
  encodes them.
* **The shared test registries are the trap.** Each needs one new entry per distribution; miss one and that suite
  silently skips your distribution, with the run still green. All five:
  [tests/property/_specs.py](tests/property/_specs.py) (`ALL_SPECS`, drives the whole property suite),
  [output_name_test.py](tests/distributions/output_name_test.py),
  [value_arg_str_test.py](tests/distributions/value_arg_str_test.py),
  [value_keyed_fast_path_test.py](tests/distributions/value_keyed_fast_path_test.py) and
  [moment_fast_path_test.py](tests/distributions/moment_fast_path_test.py) (scalar-vs-column validation contracts of
  the two fast paths, including invalid-parameter cases).
* **Write the scipy-parity test first**, then the implementation, then iterate until parity passes within tolerance.

## Common pitfalls

* **Row-alignment of scalar params.** Scalars must be coerced via `pl.repeat(value, n=pl.len(), dtype=pl.Float64())`,
   not `pl.lit(value)`. The reason: `is_elementwise=True` plugins under `over` / `group_by` expect length to track the
   partition. A bare `pl.lit` breaks this. This applies to the general (per-row) plugins; the sampler's
   constant-parameter fast path sidesteps it by passing the scalars in `kwargs` rather than as inputs.

* **The constant-parameter sampler fast path must match the per-row path bit for bit.** `<name>_sample_scalar` and the
   general sampler share `(root_seed, row_index)` seeding and the same draw, so for one seed they must agree. The fast
   path is the *only* place a distribution parameter rides in `kwargs`: a wrong field name or argument order silently
   feeds the wrong constant. `test_sample_scalar_fast_path_matches_per_row` (via the `make` vs `make_columns` spec
   builders) is what catches that; keep both builders in sync.

* **Method value arguments accept column names.** The base coerces a value / quantile arg with `as_expr`: a `str` means

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [FBruzzesi/polars-stats](https://github.com/FBruzzesi/polars-stats) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-19 -->
