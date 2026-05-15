---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

bayesDREAM is a Bayesian framework for modeling perturbation effects across multiple molecular modalities. The model consists of three sequential steps:

1. **Technical fit** (`fit_technical`): Models technical variation in non-targeting controls (NTC) to estimate gene-specific overdispersion parameters (`alpha_y`)
2. **Cis fit** (`fit_cis`): Models direct effects on the targeted gene expression (`model_x`)
3. **Trans fit** (`fit_trans`): Models downstream effects on other genes as a function of the cis gene expression (`model_y`)

The codebase uses PyTorch and Pyro for probabilistic programming and variational inference.

## Repository Structure

```
bayesDREAM_forClaude/
├── bayesDREAM/
│   ├── __init__.py          # Package exports
│   ├── model.py             # Main bayesDREAM class (~311 lines)
│   ├── core.py              # _BayesDREAMCore base class (~909 lines)
│   ├── modality.py          # Modality class for multi-modal data
│   ├── distributions.py     # Distribution-specific observation samplers
│   ├── splicing.py          # Splicing data processing (pure Python)
│   ├── fitting/             # Fitting methods (modular)
│   │   ├── __init__.py
│   │   ├── helpers.py       # Shared helper functions
│   │   ├── technical.py     # TechnicalFitter class
│   │   ├── cis.py           # CisFitter class
│   │   └── trans.py         # TransFitter class
│   ├── io/                  # Save/load functionality
│   │   ├── __init__.py
│   │   ├── save.py          # ModelSaver class
│   │   └── load.py          # ModelLoader class
│   └── modalities/          # Modality-specific methods
│       ├── __init__.py
│       ├── transcript.py    # TranscriptModalityMixin
│       ├── splicing_modality.py  # SplicingModalityMixin
│       ├── atac.py          # ATACModalityMixin
│       └── custom.py        # CustomModalityMixin
├── tests/                   # Test suite
├── toydata/                 # Test datasets (genes, splicing, metadata)
└── docs/                    # Documentation
```

**Note**: The codebase was recently refactored from a single 4,537-line `model.py` file into a modular structure. This improves maintainability while preserving backward compatibility. See `docs/archive/planning/REFACTORING_SUMMARY.md` for details.

## Core Architecture

### bayesDREAM Class

The main class in `bayesDREAM/model.py` implements multi-modal Bayesian modeling with the three-step pipeline:

**Initialization:**
- Takes cell metadata DataFrame (`meta`) with columns: `cell`, `guide`, `cell_line`, `target`, `sum_factor`, etc.
- Takes counts DataFrame (`counts`) with genes as rows, cell barcodes as columns
- Optionally takes gene metadata DataFrame (`gene_meta`) with gene annotations
  - Recommended columns: `gene`, `gene_name`, `gene_id`
  - If not provided, minimal metadata is created from counts.index
  - Flexible identifier support: uses 'gene', 'gene_name', 'gene_id', or index
- Creates guide-level metadata by grouping cells by guide and specified covariates
- Supports both CPU and CUDA devices

**Key Methods:**

- `set_technical_groups(covariates)`: Sets technical_group_code based on covariates (must be called before fit_technical)
- `fit_technical(sum_factor_col, modality_name, ...)`: Fits NTC-only model to estimate `alpha_y_prefit`
- `set_alpha_x(alpha_x, is_posterior, covariates)`: Sets cis gene overdispersion parameters
- `set_alpha_y(alpha_y, is_posterior, covariates)`: Sets trans gene overdispersion parameters
- `adjust_ntc_sum_factor(covariates, ...)`: Adjusts NTC sum factors for covariates
- `fit_cis(sum_factor_col, ...)`: Fits cis effects using `_model_x`
- `set_x_true(x_true, is_posterior)`: Sets true cis expression for trans modeling
- `permute_genes(genes2permute, ...)`: Permutes guide-gene associations for null testing
- `refit_sumfactor(covariates, ...)`: Re-estimates sum factors based on posterior cis expression
- `fit_trans(sum_factor_col, function_type, modality_name, ...)`: Fits trans effects using `_model_y`

**Function Types for Trans Modeling:**

The `fit_trans` method supports multiple functional forms for modeling how trans gene expression depends on cis gene expression:

- `single_hill`: Single Hill equation (positive or negative)
- `additive_hill`: Additive combination of positive and negative Hill functions
- `polynomial`: Polynomial function with configurable degree (default: 6)

### Probabilistic Models

Three Pyro models implement the statistical framework:

1. **`_model_technical`**: Models NTC cells to estimate baseline overdispersion
   - Negative binomial likelihood with log-normal priors
   - Estimates per-gene `alpha_y` parameters

2. **`_model_x`**: Models cis effects on the targeted gene
   - Accounts for guide-level and cell-line-level variation
   - Estimates true gene expression `x_true` for each guide
   - Uses sum factors for normalization

3. **`_model_y`**: Models trans effects as functions of cis expression
   - Supports Hill-based functions or polynomials
   - Includes sparsity priors (gamma distribution on effect sizes)
   - Models gene-specific dose-response curves

## Common Development Tasks


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [leahrosen/bayesDREAM](https://github.com/leahrosen/bayesDREAM) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
