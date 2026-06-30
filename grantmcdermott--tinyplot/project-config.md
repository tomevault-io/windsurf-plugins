---
trigger: always_on
description: **tinyplot** is a lightweight extension of base R graphics providing automatic legends, facets, themes, and other enhancements. Zero recursive dependencies — only base R.
---

# tinyplot - AI Assistant Context

## Package Overview

**tinyplot** is a lightweight extension of base R graphics providing automatic legends, facets, themes, and other enhancements. Zero recursive dependencies — only base R.

- Main function: `tinyplot()` (alias: `plt()`)
- Add layers: `tinyplot_add()` / `plt_add()`
- Themes: `tinytheme()`
- Parameters: `tpar()`

## Quick Reference

When working on tinyplot interactively, always use `pkgload::load_all()` to load the development version — never `library(tinyplot)`. This ensures you're testing your local changes, not an installed copy.

```r
pkgload::load_all()

# Then test interactively, e.g.
plt(Sepal.Length ~ Petal.Length | Species, data = iris)
```

## Repository Structure

- `R/` — Package source. The main entry point is `tinyplot.R` (~57KB). Plot types live in `type_*.R` files. Other key files include `legend.R`, `facet.R`, `by_aesthetics.R`, `tinytheme.R`, `tpar.R`, and `environment.R`. Input validation helpers follow the `sanitize_*.R` naming convention. Utility functions are in `utils.R`.
- `inst/tinytest/` — Test suite (`tinytest` + `tinysnapshot`). Snapshot SVGs are in `_tinysnapshot/`.
- `man/` — roxygen2-generated `.Rd` files.
- `vignettes/` — Package vignettes (qmd format).
- `SCRATCH/` — Developer scratch files and experiments (not part of the package).

## Code Style & Conventions

### Zero Dependency Requirement
tinyplot has zero recursive dependencies — it imports only base R packages (`graphics`, `grDevices`, `stats`, `tools`, `utils`). All contributions must preserve this. Do not add new package dependencies under `Imports` or `Depends`.

### Assignment & Syntax
```r
# Use = not <-
x = 5

# Use function() not \() — package requires R >= 4.0.0 compatibility
fn = function(x) x^2

# Prefer [[ over $ for element access (no partial matching, works with variables)
legend_args[["title"]]
settings[["datapoints"]]
# NOT: legend_args$title, settings$datapoints
```

### No Pipes in Package Code
The package targets R >= 4.0.0, so the base pipe `|>` (introduced in 4.1) is not available. Use intermediate variables or nested calls instead.

### Line Length
Wrap at ~80 characters. Break long function calls across lines.

## Architecture & Key Patterns

### Execution Flow
The main pipeline in `tinyplot.default()` follows this sequence:
1. Save par state and the call (for `tinyplot_add()` replay)
2. Build a `settings` environment with all inputs
3. Sanitize inputs: type → axes → labels → facets → datapoints
4. Run the type's `data` function (`type_data(settings)`) to transform data
5. Handle flipping, bubble sizing, axis limits
6. Compute group aesthetics (colours, pch, lty, etc.)
7. Prepare legends
8. Draw the facet grid (if any)
9. **Nested drawing loop**: outer loop over facets, inner loop over `by` groups — each iteration calls the type's `draw` function with per-group data
10. Save end par state for layer recall

### Type System
Each plot type is a `tinyplot_type` S3 object created by a `type_*()` constructor:
```r
type_boxplot = function(range = 1.5, ...) {
  out = list(
    draw = draw_boxplot(range = range, ...),  # closure: does actual plotting per group
    data = data_boxplot(...),                  # closure: preprocesses data, injects into settings
    name = "boxplot"                           # string identifier
  )
  class(out) = "tinyplot_type"
  return(out)
}
```

- `draw` function signature: `function(iby, ix, iy, ipch, ilty, icol, ibg, ...)`
  - Called once per group (`iby` = group index)
  - Receives per-group subsetted data
- `data` function signature: `function(settings, ...)`
  - Receives the `settings` environment
  - Reads from settings via `env2env(settings, environment(), keys)`
  - Writes back via `env2env(environment(), settings, keys)`
  - Can modify `datapoints`, `xlabs`, `col`, `bg`, `by`, `facet`, `group_offsets`, legend args, etc.

### Settings Environment
Individual `tinyplot()` calls store plot state in a temporary `settings` environment. Type-specific `data` functions read/write to this environment using `env2env()`. This avoids copying large objects and allows types to customize behaviour.

### Package-Level State (.tinyplot_env)
Managed via `get_environment_variable()` / `set_environment_variable()` in `environment.R`:
- `.last_call` — last tinyplot call (used by `tinyplot_add()`)
- `.saved_par_before` / `.saved_par_after` / `.saved_par_first` — par state for layer restoration
- `.tpar_hooks` — theme hooks
- `.group_offsets` — dodge offsets for layering (used by jitter-on-boxplot etc.)

### recordGraphics() for Resize Handling
Coordinate-dependent calculations (especially legends) must be wrapped in `recordGraphics()` so they replay correctly on device resize:
```r
recordGraphics(
  tinylegend(legend_env),
  list = list(legend_env = legend_env),
  env = getNamespace("tinyplot")
)
```

### Theme System
Themes use `before.plot.new` hooks. Legend code must preserve/restore hooks:
```r
oldhook = getHook("before.plot.new")
setHook("before.plot.new", function() par(new = TRUE), action = "append")
plot.new()
setHook("before.plot.new", oldhook, action = "replace")
```

### Legend Positioning

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [grantmcdermott/tinyplot](https://github.com/grantmcdermott/tinyplot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
