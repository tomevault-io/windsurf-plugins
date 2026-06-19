---
trigger: always_on
description: RandomWalker is an R package designed to generate random walks of various types that are compatible with the tidyverse ecosystem. The package provides a comprehensive set of functions for creating different types of random walks (normal, Brownian motion, geometric Brownian motion, discrete walks, and many distribution-based walks) with built-in visualization and summary capabilities.
---

# Copilot Instructions for RandomWalker

## Repository Summary

RandomWalker is an R package designed to generate random walks of various types that are compatible with the tidyverse ecosystem. The package provides a comprehensive set of functions for creating different types of random walks (normal, Brownian motion, geometric Brownian motion, discrete walks, and many distribution-based walks) with built-in visualization and summary capabilities.

**Key Purpose**: Enable users to easily generate, visualize, and analyze random walks using tidyverse-compatible functions.

## High-Level Repository Information

- **Repository Type**: R package (published on CRAN)
- **Primary Language**: R (100%)
- **Package Size**: Medium (~6,300 lines of R code across 37 files)
- **Development Stage**: Stable (version 1.0.0+)
- **Target Environment**: R (>= 4.1.0)
- **Key Dependencies**: 
  - Core: dplyr, tidyr, purrr, rlang (tidyverse ecosystem)
  - Visualization: ggplot2, ggiraph, patchwork
  - Statistics: NNS package
  - Documentation: knitr, rmarkdown
- **Distribution**: Available on CRAN as 'RandomWalker'

## Build and Validation Instructions

### Environment Setup
**ALWAYS** ensure you have R (>= 4.1.0) installed before beginning development. Install required development tools:

```r
# Install development dependencies
install.packages(c("devtools", "roxygen2", "pkgdown", "knitr", "rmarkdown"))

# Install package dependencies
install.packages(c("dplyr", "tidyr", "purrr", "rlang", "ggplot2", "ggiraph", "patchwork", "NNS"))
```

### Core Development Workflow

**ALWAYS** follow this sequence for any changes to the package:

1. **Document Functions** (ALWAYS required after function changes):
   ```r
   devtools::document()
   ```
   - Generates man/ documentation files from roxygen2 comments
   - Updates NAMESPACE file with exports/imports
   - Must be run after any function signature or documentation changes

2. **Build Package** (ALWAYS check before committing):
   ```r
   devtools::build()
   # OR from command line:
   R CMD build .
   ```
   - Creates a source package (.tar.gz)
   - Validates package structure
   - Typical build time: 30-60 seconds

3. **Check Package** (ALWAYS run for validation):
   ```r
   devtools::check()
   # OR from command line:
   R CMD check --as-cran RandomWalker_*.tar.gz
   ```
   - Runs comprehensive package validation
   - Checks documentation, examples, dependencies
   - **Target**: 0 errors, 0 warnings, minimal notes
   - Typical check time: 2-5 minutes

4. **Install Locally** (for testing):
   ```r
   devtools::install()
   ```

### Documentation and Website

**README Generation** (ALWAYS update README.Rmd, not README.md):
```r
# Edit README.Rmd, then generate README.md
rmarkdown::render("README.Rmd")
```

**Website Generation** (after significant changes):
```r
pkgdown::build_site()
```
- Generates docs/ directory with website
- Takes 1-2 minutes to complete
- Only needed for major updates or new functions

### Testing and Validation

**Note**: This package currently has no formal test suite (no tests/ directory).

**Manual Testing Approach**:
1. Load the package: `devtools::load_all()`
2. Test core functions manually:
   ```r
   library(RandomWalker)
   
   # Test basic functionality
   result <- rw30()
   head(result, 10)
   
   # Test visualization
   result |> visualize_walks()
   
   # Test summary functions
   result |> summarize_walks(.value = y)
   ```

**Example Validation** (ALWAYS run examples after changes):
```r
devtools::run_examples()
```

### Common Build Issues and Solutions

**Issue**: "Non-ASCII characters in R code"
- **Solution**: Ensure all R files use UTF-8 encoding
- Check with: `tools::showNonASCII(readLines("R/filename.R"))`

**Issue**: "Undeclared global variables"
- **Solution**: Add variables to `R/00_global_variables.R`
- Pattern: `globalVariables(c("variable_name"))`

**Issue**: Documentation not updating
- **Solution**: Delete man/ files and run `devtools::document()` again
- Ensure roxygen2 comments use proper format

## Project Layout and Architecture

### Directory Structure
```
RandomWalker/
├── .github/
│   └── ISSUE_TEMPLATE/          # Bug report and feature request templates
├── R/                           # All R source code (37 files, ~6,300 lines)
│   ├── 00_global_variables.R    # Global variable declarations for R CMD check
│   ├── auto-rw30.R             # Automatic 30-walk generator
│   ├── gen-*.R                 # Generator functions (25+ files)
│   ├── helpers.R               # Utility functions
│   ├── plt-visualize-walks.R   # Visualization functions
│   ├── stats-walk-summary.R    # Summary statistics
│   ├── vec-*.R                 # Vector utility functions
│   └── zzz.R                   # Package startup messages
├── man/                        # Generated documentation (DO NOT EDIT)
├── vignettes/
│   └── getting-started.Rmd     # Main package tutorial

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [spsanderson/RandomWalker](https://github.com/spsanderson/RandomWalker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-19 -->
