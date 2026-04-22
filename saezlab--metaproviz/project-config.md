---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Important Note on Code Quality

This package was initially developed by a beginner programmer and consequently has several areas requiring improvement:

- **Coding style** - Inconsistent formatting, naming conventions, and R idioms
- **Code quality** - Variable naming issues, redundant code, suboptimal patterns
- **Code organization** - Some functions are overly complex or poorly structured
- **API design** - Inconsistent parameter patterns, unclear function interfaces

Many of these issues can be addressed through standard refactoring and application of R best practices. When working on this codebase, prioritize:
1. **Correctness** - Fix bugs and ensure functionality works as expected
2. **Consistency** - Apply R community conventions (tidyverse style guide, etc.)
3. **Clarity** - Improve readability and maintainability
4. **Simplicity** - Reduce complexity where possible

Be prepared for ongoing refactoring tasks to improve the overall code quality.

## Project Overview

MetaProViz is a Bioconductor R package for **METabolomics pre-PRocessing, functiOnal analysis and VIZualisation**. It processes both intracellular metabolomics and exometabolomics (consumption-release/CoRe) data through five integrated modules:

1. **Processing** - Feature filtering, missing value imputation, normalization, outlier detection
2. **Differential Metabolite Analysis (DMA)** - Statistical comparison between conditions
3. **Functional Analysis** - Metabolite clustering and over-representation analysis (ORA)
4. **Prior Knowledge** - Access and refactoring of KEGG, RaMP, and other metabolic databases
5. **Visualization** - Publication-ready plots (PCA, heatmap, volcano, superplots, upset plots)

## Essential Commands

### Building and Checking

```bash
# Build the package tarball
R CMD build .

# Check the package (standard R package checks)
R CMD check *.tar.gz --no-manual

# Run BiocCheck (Bioconductor-specific checks)
Rscript -e 'BiocManager::install("BiocCheck")'
Rscript -e 'BiocCheck::BiocCheck(list.files(pattern = "*.tar.gz"))'
```

### Testing

```bash
# Run all tests
Rscript -e 'devtools::test()'

# Run specific test file
Rscript -e 'testthat::test_file("tests/testthat/test-checkmatch_pk_to_data.R")'

# Run tests with coverage
Rscript -e 'covr::package_coverage()'
```

### Installation

```bash
# Install from local source
Rscript -e 'devtools::install()'

# Install with dependencies
Rscript -e 'devtools::install_deps(dependencies = TRUE)'

# Install OmnipathR from Bioconductor devel (required dependency)
Rscript -e 'if (!requireNamespace("BiocManager", quietly = TRUE)) install.packages("BiocManager")'
Rscript -e 'BiocManager::install(version = "devel")'
Rscript -e 'BiocManager::install("OmnipathR")'
```

### Documentation

```bash
# Build documentation
Rscript -e 'devtools::document()'

# Build vignettes
Rscript -e 'devtools::build_vignettes()'

# Build pkgdown site
Rscript -e 'pkgdown::build_site()'
```

### Loading and Debugging

```bash
# Load package for development
Rscript -e 'devtools::load_all()'

# Check for code style issues
Rscript -e 'lintr::lint_package()'
```

## Code Architecture

### Module Organization

The codebase is organized into distinct functional modules in `/R`:

**Core Analysis Modules:**
- `Processing.R` - Data preprocessing pipeline (filtering, imputation, normalization, outlier detection)
- `DifferentialMetaboliteAnalysis.R` - `dma()` function for statistical comparison
- `MetaboliteClusteringAnalysis.R` - `mca_2cond()`, `mca_3cond()` for clustering by regulatory rules
- `MetaDataAnalysis.R` - `metadata_analysis()` for PCA and ANOVA on sample metadata
- `OverRepresentationAnalysis.R` - `cluster_ora()` for pathway enrichment analysis

**Prior Knowledge:**
- `GetPriorKnowledge.R` - Functions to load KEGG, RaMP pathways (`metsigdb_*` functions)
- `RefactorPriorKnoweldge.R` - `translate_id()` for ID translation between databases

**Visualization (Viz* files):**
- `VizVolcano.R`, `VizHeatmap.R`, `VizPCA.R` - Standard omics plots
- `VizSuperplots.R` - Box/violin/bar plots with statistical overlays
- `VizStackedbar.R`, `VizUpset.R` - Advanced visualization types
- `HelperPlots.R` - Shared plotting utilities and themes

**Infrastructure (Helper* files):**
- `HelperChecks.R` - Input validation for all functions
- `HelperLog.R` - Logging system (wraps OmnipathR/logger)
- `HelperSave.R` - File I/O for tables and plots
- `HelperOptions.R` - Configuration management
- `HelperMisc.R` - Utility functions

**Special files:**
- `aaa_MetaProViz_Env.R` - Global environment for state management (loaded first)
- `zzz.R` - Package initialization hooks (loaded last)
- `enricher_internal.R` - Internal copy of DOSE package enrichment logic

### Data Flow Pattern

All functions follow a standardized input/output pattern:

**Inputs:**
- `data` - Data frame with samples as rows, metabolites as columns
- `metadata_sample` - Sample metadata with `Conditions` column (required)
- `metadata_feature` - Feature/metabolite metadata (optional)
- `metadata_info` - Named vector for flexible parameter specification

**Outputs:**
```r
list(

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/saezlab) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
