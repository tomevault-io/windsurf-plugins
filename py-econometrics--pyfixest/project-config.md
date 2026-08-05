---
trigger: always_on
description: pyfixest ports R's `fixest` to Python: high-dimensional fixed-effects estimation
---

# pyfixest — guide for coding agents

pyfixest ports R's `fixest` to Python: high-dimensional fixed-effects estimation
(OLS/WLS, IV, Poisson, GLM, quantile regression) with fixest formula syntax, a
post-estimation toolbox, and Rust kernels for hot loops.

Two rules beat everything else:

1. **Mirror `fixest`** in user-facing behavior, naming, and defaults, unless there
   is a documented reason not to.
2. **Mirror the nearest existing implementation.** Almost every kind of change has
   an in-repo precedent. Find it and copy its structure before writing new code.

For the end-to-end workflow (implement a feature, clean up a contributor PR),
follow **`.agents/feature-pr.md`**. This file is the tool-neutral entry point:
Claude Code (via `CLAUDE.md`), Codex, and OpenCode read `AGENTS.md` natively;
if a tool in your setup does not, point its rules/config file here instead of
duplicating content. `CLAUDE.md` is a thin redirect (`@AGENTS.md`) and is
committed to the repo.
Edit the workflow only in `.agents/feature-pr.md`.

## Repo map

| Path | Contents |
|---|---|
| `pyfixest/estimation/api/` | Public entry points, one module per function: `feols`, `fepois`, `feglm`, `quantreg`; shared input checks in `api/utils.py` |
| `pyfixest/estimation/models/` | Model/result classes (`Feols`, `Feiv`, `Fepois`, `Feglm`, …); modules end in `_` (`feols_.py`) |
| `pyfixest/estimation/internals/` | Shared estimation internals: `vcov_utils.py`, `solvers.py`, `collinearity.py`, `separation.py`, `literals.py` |
| `pyfixest/estimation/post_estimation/` | Post-estimation features (`ritest`, `ccv`, `decomposition`, `prediction`, `multcomp`); model classes hold only wrapper methods free of numerics |
| `pyfixest/estimation/formula/` | Formula parsing and model-matrix construction (wraps formulaic) |
| `pyfixest/estimation/` root | `config.py` (`EstimationConfig`), `plan_.py` (`parse_formula`, `fit_one`), `runner.py`, `FixestMulti_.py` (pure container for multiple estimation); backend subpackages `numba/` and `torch/`; `deprecated/`. Root-level `feols_.py`/`feiv_.py`/`fepois_.py` are compat shims — the real classes live in `models/` |
| `pyfixest/demeaners.py` | Public demeaner configs (`BaseDemeaner`, `MapDemeaner`, `LsmrDemeaner`) behind the `demeaner=` argument; own quartodoc section |
| `pyfixest/core/` | Python wrappers and `_core_impl.pyi` type stubs for the Rust extension |
| `src/` | Rust kernels (PyO3), registered in `src/lib.rs` |
| `pyfixest/did/`, `pyfixest/report/`, `pyfixest/utils/` | DiD estimators; `etable`/plots; data utilities and DGPs (`utils/dgps.py`) |
| `tests/` | Pytest suite; reference scripts and stored outputs in `tests/data/` |
| `docs/` | Quarto site, Diataxis layout: `tutorials/`, `how-to/`, `explanation/`, `textbook-replications/` |

Estimation flow: `feols()` builds an `EstimationConfig` → `parse_formula`
(`plan_.py`) expands multiple-estimation syntax → `FixestMulti` holds the models →
`runner.run_estimation` / `plan_.fit_one` drive each model through
`prepare_model_matrix → get_fit → vcov → get_inference`, where `get_fit` runs
`demean → to_array → drop_multicol_vars → wls_transform` before solving.
Post-estimation happens via methods on the fitted model.

## Where new code goes

**New post-estimation feature** — standalone module in
`estimation/post_estimation/` holding the logic; a thin method on `Feols` (and
siblings where applicable) that validates inputs and delegates. The commit history
is one long effort to carve logic *out* of the model classes — do not grow them.
Template: `post_estimation/ritest.py` + `Feols.ritest`.

**New vcov type** — add the option to `internals/literals.py`; accept and
validate it in `_check_vcov_input` / `_deparse_vcov_input`
(`models/feols_.py`); dispatch from `Feols.vcov()` via a small `_vcov_<name>`
method; put the meat/bread math in `internals/vcov_utils.py` (or a Rust kernel);
thread through `FixestMulti.vcov()` and quantreg if applicable; wire ssc via
`_make_ssc_kwargs`. Template: the NW/DK HAC path.

**New estimation entry point** — own module in `estimation/api/`; export through
`estimation/api/__init__.py` and `pyfixest/__init__.py` (`__all__`,
`_lazy_imports`, `_direct_module_imports`); add to the quartodoc `contents` list
in `docs/_quarto.yml`. Keep the signature order consistent with siblings:
`fml, data, vcov, …, copy_data, store_data, lean, …`. User-facing functions that
are *not* estimation entry points (`rwolf`, `bonferroni`, `wyoung`) instead live
in `post_estimation/` and are exported through `estimation/__init__.py` plus the
top-level `_lazy_imports`.

**New Rust kernel** — `src/<topic>.rs` with a function named `_<name>_rs`;
register in `src/lib.rs`; add a typed stub to `pyfixest/core/_core_impl.pyi`;
re-export under a clean alias in `pyfixest/core/<topic>.py`. Keep a NumPy
reference implementation around for tests when feasible. Template: `src/nw.rs` →
`pyfixest/core/nw.py`.

**New estimation-time option** — `Literal` alias in `internals/literals.py`;
accept it in each relevant `api/` function with a documented default; validate
early (`_validate_literal_argument` or `api/utils._estimation_input_checks`);
thread through `EstimationConfig` and `plan_._build_model_kwargs`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [py-econometrics/pyfixest](https://github.com/py-econometrics/pyfixest) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->
