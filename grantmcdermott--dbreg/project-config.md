---
trigger: always_on
description: **dbreg** is an R package for fast regressions on database backends, designed for datasets too large for R's memory. Uses DuckDB as the default backend with support for other DBI-compatible databases. Zero non-essential dependencies in Imports.
---

# dbreg - AI Assistant Context

## Package Overview

**dbreg** is an R package for fast regressions on database backends, designed for datasets too large for R's memory. Uses DuckDB as the default backend with support for other DBI-compatible databases. Zero non-essential dependencies in Imports.

- Main function: `dbreg()` — regression with automatic strategy selection
- Binscatter: `dbbinsreg()` — binned scatter plots on database backends
- SQL design matrix: `sql_model_matrix()` — factor/interaction expansion to SQL

## Quick Reference

Always use `pkgload::load_all()` for development — never `library(dbreg)`.

```r
pkgload::load_all()

# Then test interactively, e.g.
dbreg(weight ~ Time | Diet, data = ChickWeight)
dbreg(weight ~ Time | Diet, data = ChickWeight, vcov = "hc1")
dbreg(weight ~ Time | Diet + Chick, data = ChickWeight, strategy = "mundlak")
```

## Repository Structure

- `R/` — Package source:
  - `dbreg.R` — public API, input processing, strategy selection, alternating projections, finalization
  - `strategies.R` — strategy execution functions (called from `dbreg.R`)
  - `vcov.R` — variance-covariance and meat matrix computation (shared across strategies)
  - `utils.R` — shared helpers: SQL dialect, formula parsing, connection setup, `env2env`, `gen_xvar_pairs`
  - `dbbinsreg.R` — binscatter on database backends
  - `sql_model_matrix.R` — factor/interaction expansion to SQL
  - `stats-methods.R`, `tidiers.R`, `print.R`, `gof.R`, `plot.r` — S3 methods and output
- `inst/tinytest/` — Test suite (tinytest framework).
- `man/` — roxygen2-generated `.Rd` files.
- `vignettes/` — Package vignette (`intro.qmd`).
- `_benchmarks/` — Performance comparisons with plots.
- `nyc-taxi/` — Large test dataset (180M rows, Hive-partitioned parquet). Not in repo; see download instructions below.
- `SCRATCH/` — Developer scratch files and experiments (not part of the package).

## Code Style & Conventions

### Assignment & Syntax
```r
# Use = not <-
x = 5

# Prefer explicit function() (not \() for broader compatibility)
fn = function(x) x^2

# Prefer [[ over $ for element access (no partial matching, works with variables)
inputs[["yvar"]]
result[["coeftable"]]
# NOT: inputs$yvar, result$coeftable
```

### Dependency Policy
Minimize Imports — avoid adding new dependencies unless truly necessary. Non-essential packages go in Suggests/Enhances. All heavy computation should happen in SQL, not R.

### SQL Design
Build modular, reusable SQL-generating helpers (e.g., `sql_weighted_mean()`, `sql_weighted_sum()`, `sql_count()`, `build_weighted_moment_terms()`) that compose into larger queries. Strategy functions (`execute_*_strategy()`) assemble these building blocks into full CTEs rather than writing bespoke SQL inline. This keeps the code DRY across strategies that share common patterns (e.g., weighted vs unweighted moment computation).

### Line Length
No strict limit, but keep lines readable. Break long SQL strings with `paste0()` or `glue()`.

## Architecture

### Execution Flow (`dbreg()`)
1. `process_dbreg_inputs()` — validate args, set up DB connection, parse formula, filter missings, validate weights. Returns an **environment** (not a list).
2. `choose_strategy(inputs)` — auto-selection logic. Mutates `inputs[["is_balanced"]]` and `inputs[["compression_ratio_est"]]` in place (reference semantics).
3. `execute_*_strategy(inputs)` — one of: `moments`, `demean`, `mundlak`, `compress`
4. `finalize_dbreg_result(result, inputs, chosen_strategy)` — set class, attach metadata

### The `inputs` Environment
`inputs` is an environment (created via `list2env()`) that flows through the pipeline. Using an environment rather than a list gives reference semantics — functions like `choose_strategy()` can store computed metadata (e.g., `is_balanced`, `compression_ratio_est`) that downstream functions read without needing return-value plumbing.

Strategy functions access fields via `inputs[["field"]]` and typically extract frequently-used values into local variables at the top of the function for readability.

Both `dbreg()` and `dbbinsreg()` follow this pattern.

### Acceleration Strategies
1. **compress** — GROUP BY compression → frequency-weighted least squares (Wong et al. 2021). Best when regressors are discrete.
2. **moments** — Direct sufficient statistics (X'X, X'y) via SQL aggregation. No FE. Single-row result.
3. **demean/within** — Demeaning for 1+ FE. Analytic for 1 FE or balanced 2-way panels; alternating projections (AP) for weighted, unbalanced, or 3+ FE cases.
4. **mundlak** — True Mundlak/CRE estimator: Y ~ X + group means of X. Any number of FE, any panel structure. Single-pass (no iteration).

### Auto Strategy Logic
- No FE + (continuous vars OR poor compression) → `"moments"`
- 1 FE + poor compression → `"demean"`
- 2 FE + poor compression + balanced → `"demean"` (analytic double demeaning)
- 2 FE + poor compression + unbalanced → `"demean"` (alternating projections)
- 3+ FE + poor compression → `"demean"` (alternating projections)
- Otherwise → `"compress"`

### Variance-Covariance Computation

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [grantmcdermott/dbreg](https://github.com/grantmcdermott/dbreg) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-02 -->
