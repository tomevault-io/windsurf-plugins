---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Environment Setup

**IMPORTANT**: Always activate the virtual environment before running Python or git operations:

```bash
source .venv/bin/activate
```
If working in a worktree, make sure there's a local `.venv` first by running `uv sync` in the worktree directory. Do NOT `cd` to the main repo — all commands (including git) should run in the worktree.

Repo requires `.env` file with WandB credentials (see `.env.example`)

## Project Overview

PD is a research framework for analyzing neural network components and their interactions through sparse parameter decomposition techniques.

- Target model parameters are decomposed as a sum of `parameter components`
- Parameter components approximate target model outputs despite differentiable stochastic masks
- Causal importance functions quantify how much each component can be masked on each datapoint
- Multiple loss terms balance faithfulness, output reconstruction quality, and component activation sparsity

The codebase supports three experimental domains: TMS (Toy Model of Superposition), ResidualMLP (residual MLP analysis), and Language Models.

**Available experiments** (defined in `param_decomp/registry.py`):

- **TMS (Toy Model of Superposition)**:
  - `tms_5-2` - TMS with 5 features, 2 hidden dimensions
  - `tms_5-2-id` - TMS with 5 features, 2 hidden dimensions (fixed identity in-between)
  - `tms_40-10`
  - `tms_40-10-id`
- **ResidualMLP**:
  - `resid_mlp1` - 1 layer
  - `resid_mlp2` - 2 layers
  - `resid_mlp3` - 3 layers
- **Language Models** (MLP-only Llama variants):
  - `ss_llama_simple_mlp-2L` - 2-layer Llama on SimpleStories
  - `pile_llama_simple_mlp-4L` - 4-layer Llama on the Pile (the VPD-paper run)
  - `pile_llama_simple_mlp-12L` - 12-layer Llama on the Pile

The `lm` experiment can decompose any HuggingFace-loadable model whose target modules are
`nn.Linear`, `nn.Embedding`, or `transformers.modeling_utils.Conv1D`.

## Research Papers

This repository implements methods from two key research papers on parameter decomposition:

**Stochastic Parameter Decomposition (SPD)**

- [`papers/Stochastic_Parameter_Decomposition/spd_paper.md`](papers/Stochastic_Parameter_Decomposition/spd_paper.md)
- A version of this repository was used to run the experiments in this paper. But we continue to develop on the code, so it no longer is limited to the implementation used for this paper.
- Introduces the core SPD framework
- Details the stochastic masking approach and optimization techniques used throughout the codebase
- Useful reading for understanding the implementation details, though may be outdated.

**Attribution-based Parameter Decomposition (APD)**

- [`papers/Attribution_based_Parameter_Decomposition/apd_paper.md`](papers/Attribution_based_Parameter_Decomposition/apd_paper.md)
- This paper was the precursor to SPD.
- It introduced the concept of linear parameter decomposition.
- Contains theoretical foundations, broader context, and high-level conceptual insights of parameter decomposition methods.
- Useful for understanding the conceptual framework and motivation behind SPD

## Development Commands

**Setup:**

- `make install-dev` - Install package with dev dependencies and pre-commit hooks
- `make install` - Install package only (`pip install -e .`)
- `make install-app` - Install frontend dependencies (`npm install` in `param_decomp/app/frontend/`)

**Code Quality:**

- `make check` - Run full pre-commit suite (basedpyright, ruff lint, ruff format)
- `make type` - Run basedpyright type checking only
- `make format` - Run ruff linter and formatter

**Frontend (when working on `param_decomp/app/frontend/`):**

- `make check-app` - Run frontend checks (format, type check, lint)
- Or run individually from `param_decomp/app/frontend/`:
  - `npm run format` - Format code with Prettier
  - `npm run check` - Run Svelte type checking
  - `npm run lint` - Run ESLint

**Testing:**

- `make test` - Run tests (excluding slow tests)
- `make test-all` - Run all tests including slow ones
- `python -m pytest tests/test_specific.py` - Run specific test file
- `python -m pytest tests/test_specific.py::test_function` - Run specific test

**Running the App:**

- `make app` - Launch the PD visualization app (backend + frontend)

## Architecture Overview

**Core PD Framework:**

- `param_decomp/run_param_decomp.py` - Main PD optimization logic called by all experiments
- `param_decomp/configs.py` - Pydantic config classes for all experiment types
- `param_decomp/registry.py` - Centralized experiment registry with all experiment configurations
- `param_decomp/models/component_model.py` - Core ComponentModel that wraps target models
- `param_decomp/models/components.py` - Component types (LinearComponent, EmbeddingComponent, etc.)
- `param_decomp/losses.py` - PD loss functions (faithfulness, reconstruction, importance minimality)
- `param_decomp/metrics.py` - Metrics for logging to WandB (e.g. CI-L0, KL divergence, etc.)
- `param_decomp/figures.py` - Figures for logging to WandB (e.g. CI histograms, Identity plots, etc.)

**Terminology: Sources vs Masks:**


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [goodfire-ai/param-decomp](https://github.com/goodfire-ai/param-decomp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
