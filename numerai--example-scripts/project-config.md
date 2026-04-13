---
trigger: always_on
description: This repository contains example scripts and an agentic research framework for the [Numerai data science tournaments](https://numer.ai/). AI coding agents can use this file to understand how to help with Numerai-related tasks.
---

# AGENTS.md - AI Agent Instructions

This repository contains example scripts and an agentic research framework for the [Numerai data science tournaments](https://numer.ai/). AI coding agents can use this file to understand how to help with Numerai-related tasks.

## Quick Reference

| Task | Resource |
|------|----------|
| Run experiments | `numerai-experiment-design` skill |
| Add new model types | `numerai-model-implementation` skill |
| Create & deploy pkl files | `numerai-model-upload` skill |
| Query tournament data | Numerai MCP server |
| Upload models programmatically | Numerai MCP server |

---

## Tournament Guide

- "Numerai" is the original tournament in which you are provided anonymized and obfuscated data. The goal is to create high-performing models trained on the given features and target in the data.

- "Signals" is the second tournament in which you are expected build your own stock market dataset. You are given a basic dataset with a target and rudimentary features. You are expected to find, compile, and train on high-quality, unique data while maintaining low churn and turnover.

- "Crypto" is a toy tournament for cryptocurrency prediction. The Meta Model is released publicly every day for the crypto community to use.

## General Recommendations
- Install the Numerai MCP
- Install the following python packages:
  - numerapi
  - numerai-tools
- For heavier-duty automation using cloud infrastructure, suggest numerai-cli to the user

## Repository Structure

```
example-scripts/
├── numerai/
│   ├── agents/                    # Agentic research framework
│   │   ├── AGENTS.md              # Detailed agent instructions
│   │   ├── baselines/             # Baseline model configurations
│   │   ├── code/                  # Shared packages
│   │   │   ├── analysis/          # Reporting & plotting
│   │   │   ├── data/              # Dataset builders
│   │   │   ├── metrics/           # BMC/corr scoring utilities
│   │   │   └── modeling/          # Training pipeline & model wrappers
│   │   ├── experiments/           # Experiment results (not in git)
│   │   └── skills/                # Codex skills for agent workflows
│   └── *.ipynb                    # Tournament-specific notebooks
├── signals/                       # Signals tournament examples
├── crypto/                        # Crypto tournament examples
├── cached-pickles/                # Pre-built model pickles
```

---

## Skills Overview

The `numerai/agents/skills/` folder contains structured workflows for common tasks. Each skill has a `SKILL.md` file with detailed instructions.

### 1. `numerai-experiment-design`

**Purpose**: Design, run, and report Numerai experiments for any model idea.

**When to use**: 
- Testing a new research hypothesis
- Sweeping hyperparameters or targets
- Comparing model variants against baselines

**Key workflow**:
1. Plan the experiment (baseline, metrics, sweep dimensions)
2. Create config files in `agents/experiments/<name>/configs/`
3. Run training via `python -m agents.code.modeling --config <config>`
4. Analyze results and iterate
5. Scale winners to full data
6. Generate final report with plots

**Entry points**:
- `python -m agents.code.modeling --config <config_path>`
- `python -m agents.code.analysis.show_experiment`
- `python -m agents.code.data.build_full_datasets`

### 2. `numerai-model-implementation`

**Purpose**: Add new model types to the training pipeline.

**When to use**:
- Implementing a new ML architecture (e.g., transformers, custom ensembles)
- Adding support for a new library (e.g., XGBoost, CatBoost)
- Creating custom preprocessing or inference logic

**Key steps**:
1. Create model wrapper in `agents/code/modeling/models/`
2. Register in `agents/code/modeling/utils/model_factory.py`
3. Add config using the new model type
4. Validate with smoke test (corr_mean should be 0.005-0.04)

### 3. `numerai-model-upload`

**Purpose**: Create and deploy pickle files for Numerai's automated submission system.

**When to use**:
- Preparing a trained model for tournament submission
- Setting up automated weekly predictions
- Debugging pickle validation failures

**Critical requirements**:
- Python version must match Numerai's compute environment
- Pickle must be self-contained (no repo imports)
- `predict(live_features, live_benchmark_models)` signature required

**Workflow**:
1. Query default Docker image for Python version
2. Create matching venv with pyenv
3. Train final model and export inference bundle
4. Build self-contained `predict` function
5. Test with `numerai_predict` Docker container
6. Deploy via MCP server

---

## Numerai MCP Server

The `numerai` MCP server provides programmatic access to the Numerai Tournament API. If available, agents should use it for tournament operations.

It can be installed via curl through:

```bash
curl -sL https://numer.ai/install-mcp.sh | bash
```

This install script sets Codex CLI up with the MCP configuration as well as configures an environment variable for an MCP API key.

### Available Tools

| Tool | Purpose |
|------|---------|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/numerai) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-13 -->
