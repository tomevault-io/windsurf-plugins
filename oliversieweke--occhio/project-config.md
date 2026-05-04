---
trigger: always_on
description: [//]: # (Referenced or symlinked by: README.md, CLAUDE.md, .clinerules, .cursor/rules/AGENTS.md, .windsurf/rules/main.md, .github/copilot-instructions.md, .junie/guidelines.md, aider.conf.yml )
---

[//]: # (Referenced or symlinked by: README.md, CLAUDE.md, .clinerules, .cursor/rules/AGENTS.md, .windsurf/rules/main.md, .github/copilot-instructions.md, .junie/guidelines.md, aider.conf.yml )

# Context

`occhio` is a Python research library for experimenting with [Toy Models of Superposition](https://transformer-circuits.pub/2022/toy_model/index.html) — a mechanistic interpretability research setup where neural networks are trained to compress sparse feature distributions into lower-dimensional representations. The library provides composable building blocks (distributions, autoencoders, training loops, visualization) to make it easy to run and analyze these experiments.

# Quick Commands

- Install dependencies: `uv sync`
- Formatting: `uv run ruff format`
- Linting: `uv run ruff check`
- Clear Notebooks: `uv run jupyter nbconvert --clear-output --inplace`

# Skills

## ⚠️ CRITICAL: Pre-Implementation Check

**STOP**: Before writing ANY code or running ANY commands, check the available skills below.

| Category                   | Skill                                                            | Description                                   |
|----------------------------|------------------------------------------------------------------|-----------------------------------------------|
| Code Quality               | [formatting-and-linting](skills/formatting-and-linting/SKILL.md) | Ruff setup and usage                          |
| Documentation              | [documentation](skills/documentation/SKILL.md)                   | Sphinx docs building and live reload          |
| MCP Server                 | [mcp](skills/mcp/SKILL.md)                                       | MCP server development and tool creation      |
| Visualization and Plotting | [visualization](skills/visualization-and-plotting/SKILL.md)      | BasePlot pattern for ToyModel/ModelGrid plots |
| Meta                       | [skill-creator](skills/skill-creator/SKILL.md)                   | Create, test, and improve skills              |

Keep the skills up to date if you make changes to the codebase affecting them.

# Documentation

## Key concepts

- **`ToyModel`** ([src/occhio/toy_model.py](src/occhio/toy_model.py)): Core experiment object. Combines a `Distribution` with an `AutoEncoderBase`. Provides `.fit()`, `.sample_latent()`, `.get_one_hot_embeddings()`, and geometric analysis properties (`W`, `feature_norms`, `interferences`, `feature_dimensionalities`, etc.).
- **`ModelGrid`** ([src/occhio/model_grid.py](src/occhio/model_grid.py)): Vectorized grid of `ToyModel`s parameterized over one or more `Axis` values. Uses `torch.vmap` + `torch.compile` for fast parallel training. Supports `snapshot_interval` to capture training dynamics as a `TrainingAxis`.
- **Distributions** ([src/occhio/distributions/](src/occhio/distributions/)): All subclass `Distribution`. Must implement `.sample(batch_size) -> Tensor`. See `distributions/README.md` for a full taxonomy.
- **AutoEncoders** ([src/occhio/autoencoder.py](src/occhio/autoencoder.py)): All subclass `AutoEncoderBase(nn.Module)`. Must implement `.encode()`, `.decode()`, `.resample_weights()`, and set `self.n_features` + `self.n_hidden` in `__init__`.
- **SAEs** ([src/occhio/sae/sae.py](src/occhio/sae/sae.py)): Sparse AutoEncoders with L1 sparsity penalties, separate from the `AutoEncoderBase` hierarchy.

## Where things live

```
src/occhio/
├── __init__.py              # Exports: ToyModel, ModelGrid, AutoEncoderBase
├── toy_model.py             # ToyModel class
├── model_grid.py            # ModelGrid, Axis, TrainingAxis
├── autoencoder.py           # AutoEncoderBase, TiedLinear, TiedLinearRelu, MLPEncoder, ComputeAutoEncoder
├── distributions/
│   ├── base.py              # Distribution (ABC), DistributionStack
│   ├── sparse.py            # SparseUniform, SparseExponential, SingleUniform
│   ├── correlated.py        # CorrelatedPairs, HierarchicalPairs, ScaledHierarchicalPairs, AnticorrelatedPairs
│   ├── relational.py        # RelationalSimple, MultiRelational
│   ├── hierarchical.py      # HierarchicalSparse
│   ├── dag.py               # DAGDistribution, DAGBayesianPropagation, DAGRandomWalkToRoot, PowerLawDigraph
│   └── README.md            # Distribution taxonomy docs
├── mcp/                     # MCP server (optional, `uv sync --extra mcp`)
│   ├── server.py            # FastMCP server + CLI entrypoint
│   ├── graphql_client.py    # Shared async helpers
│   └── servers/             # Tool modules (alignmentforum.py, etc.)
├── sae/sae.py               # SAESimple, TopKIgnoreSAE, CausalSAE
├── visualization/           # Plotly-based plotting (embedding, geometry, phase change, etc.)
├── utils/device.py          # _same_device() helper
└── examples/                # Runnable example scripts and notebooks

.ai/mcp/mcp.json             # Canonical MCP server config (symlinked to .claude/, .cursor/, etc.)
experiments/                 # Ad-hoc research notebooks and scripts (not packaged)
tests/                       # Top-level pytest tests
```

## Conventions


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [OliverSieweke/occhio](https://github.com/OliverSieweke/occhio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
