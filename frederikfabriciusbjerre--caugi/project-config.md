---
trigger: always_on
description: `caugi` (pronounced "corgi") is a **Causal Graph Interface** package for R, providing a high-performance, tidy toolbox for building, coercing, and analyzing causal graphs. The package is built as a hybrid R/Rust codebase, leveraging Rust for performance-critical operations and R for the user-facing API.
---

# Agent Instructions for caugi

## Repository Overview

`caugi` (pronounced "corgi") is a **Causal Graph Interface** package for R, providing a high-performance, tidy toolbox for building, coercing, and analyzing causal graphs. The package is built as a hybrid R/Rust codebase, leveraging Rust for performance-critical operations and R for the user-facing API.

### Architecture

- **R Package**: Front-end API using S7 objects, tidyverse principles
- **Rust Backend**: Core graph algorithms implemented in Rust using the `extendr` framework
- **Graph Storage**: Compressed Sparse Row (CSR) format for efficient querying
- **Lazy Building**: Graph mutations are batched in R and built in Rust on demand

## Code Style Guidelines

### R Code

- **Follow tidyverse style guide**: Run `air format .` to format all R code before committing (configured via `air.toml`). Use `air format . --check` to check without modifying files.
- **Roxygen2 documentation**: All exported functions must have comprehensive documentation with `@title`, `@description`, `@param`, `@returns`, and `@examples`
- **Naming conventions**:
  - Functions: `snake_case`
  - S7 classes: `snake_case`
  - Internal functions: prefix with `.` (e.g., `.internal_function`)

### Rust Code

- **Follow Rust standard style**: Run `cargo fmt` before committing Rust code
- **Documentation**: Use Rust doc comments (`///`) for public functions and modules
- **Performance-focused**: Prioritize performance and memory efficiency in Rust code
- **Error handling**: Use `Result` types appropriately and provide meaningful error messages

## Project Structure

```
caugi/
├── R/                      # R source files
│   ├── caugi.R            # Main caugi object constructor
│   ├── edge_operators.R   # Edge operator definitions
│   ├── queries.R          # Graph query functions
│   ├── metrics.R          # Graph metrics (SHD, AID)
│   ├── adjustment.R       # Adjustment set functions
│   ├── DSL-parser.R       # DSL parsing logic
│   ├── format-dot.R       # DOT format output
│   ├── format-mermaid.R   # Mermaid format output
│   ├── simulation.R       # Graph simulation
│   ├── verbs.R            # Tidyverse-style verbs
│   ├── operations.R       # Graph operations
│   ├── as_caugi.R         # Coercion to caugi
│   ├── caugi_to.R         # Coercion from caugi
│   └── ...
├── src/
│   ├── rust/              # Rust source code
│   │   ├── src/
│   │   │   ├── lib.rs     # Main library and extendr bindings
│   │   │   ├── edges/     # Edge type definitions
│   │   │   └── graph/     # Graph data structures and algorithms
│   │   │       ├── alg/   # Core graph algorithms
│   │   │       ├── dag/   # DAG-specific functionality
│   │   │       ├── pdag/  # PDAG-specific functionality
│   │   │       ├── admg/  # ADMG-specific functionality
│   │   │       └── layout/ # Graph layout algorithms (Sugiyama, force-directed, etc.)
│   │   └── Cargo.toml
│   └── entrypoint.c       # C entrypoint for R
├── tests/
│   └── testthat/          # Test files (test-*.R)
├── man/                   # Generated documentation
└── vignettes/             # Package vignettes
```

## Development Workflow

### Building and Testing

1. **Build R package**: Use `devtools::load_all()` or standard R CMD build
2. **Build Rust**: Rust compilation happens automatically via `rextendr`
3. **Run tests**: `devtools::test()` or `testthat::test_local()`
4. **Check package**: `devtools::check()` runs R CMD check
5. **Code coverage**: Monitored via codecov
6. **Build documentation site**: `pkgdown::build_site()` builds the package website configured in `_pkgdown.yml`
7. **Re-vendor Rust deps**: Whenever `src/rust/Cargo.lock` changes (added, removed, or bumped Rust dependency), regenerate the offline-build cache by running `rextendr::vendor_pkgs(overwrite = TRUE)` from the package root. This refreshes `src/rust/vendor.tar.xz` and `src/rust/vendor-config.toml`; commit both alongside the `Cargo.toml`/`Cargo.lock` change.

### Testing Requirements

- **Write tests for new features**: All new functions should have corresponding tests in `tests/testthat/`
- **Test file naming**: `test-<feature>.R` (e.g., `test-queries.R`)
- **Use testthat**: Follow existing patterns with `test_that()` blocks
- **Test edge cases**: Consider empty graphs, single nodes, and complex graph structures
- **Test both R and Rust paths**: Ensure lazy building works correctly

### Making Changes

1. **Minimal changes**: Make the smallest possible changes to accomplish the goal
2. **Edge registry**: Be careful when modifying the edge registry system
3. **Backward compatibility**: Maintain API compatibility when possible
4. **Update NEWS.md**: Add entries to `NEWS.md` for user-facing changes under the appropriate section:
   - **New Features**: New functions, methods, or capabilities
   - **Improvements**: Enhancements to existing functionality, performance, or documentation
   - **Bug Fixes**: Corrections to existing behavior
   - Use bullet points starting with `*` and include function names in backticks

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [frederikfabriciusbjerre/caugi](https://github.com/frederikfabriciusbjerre/caugi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-21 -->
