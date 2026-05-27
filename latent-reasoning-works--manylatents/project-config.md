---
trigger: always_on
description: Unified dimensionality reduction and neural network analysis. PyTorch Lightning + Hydra + uv.
---

# CLAUDE.md

Unified dimensionality reduction and neural network analysis. PyTorch Lightning + Hydra + uv.

**See [ARCHITECTURE.md](ARCHITECTURE.md) for the codebase map, data flow, and architectural invariants.**

## Before Starting Work

- **Check for existing implementations** before building anything new. Run `git log --oneline main | head -30` and `gh pr list --state merged --limit 10` to avoid reimplementing features that already exist under different naming.
- **Always use manylatents APIs** (e.g. `from manylatents.api import run`) for experiments and traces — never substitute with raw sklearn/numpy equivalents unless explicitly told to.
- **Prefer minimal, scoped changes.** Do not proactively expand scope (adding config files, fixing docs, cleaning orphan configs) unless explicitly asked. Ask before doing extra work.

## Using the Python API

**Call `run()` directly with parameters. Do not wrap it.** The API is designed for functional composition — pass data, algorithms, and metrics as arguments. Do not create helper functions, wrapper classes, or "pipeline" abstractions around it.

```python
# CORRECT — direct functional call
from manylatents.api import run

result = run(
    data="swissroll",
    algorithms={"latent": "pca"},
    metrics={"trustworthiness": {
        "_target_": "manylatents.metrics.trustworthiness.Trustworthiness",
        "_partial_": True, "n_neighbors": 5,
    }},
)
result["embeddings"]  # (n, d) ndarray
result["scores"]      # {"trustworthiness": 0.95}
```

```python
# WRONG — unnecessary wrapper
def run_pca_analysis(data_path, n_components=2):
    """Don't do this."""
    module = PCAModule(n_components=n_components)
    data = load_data(data_path)
    module.fit(data)
    return module.transform(data)
```

When writing analysis scripts, parameterize through `run()` arguments or Hydra config overrides, not through custom wrapper functions. If you need to vary parameters, use a loop over `run()` calls or a Hydra multirun.

## CLI Entry Points

```bash
# CLI — primary interface
uv run python -m manylatents.main algorithms/latent=pca data=swissroll metrics=trustworthiness

# LightningModule path
uv run python -m manylatents.main algorithms/lightning=ae_reconstruction data=swissroll trainer.fast_dev_run=true

# Multirun sweep
uv run python -m manylatents.main --multirun algorithms/latent=umap,phate,tsne data=swissroll metrics=trustworthiness

# SLURM submission
uv run python -m manylatents.main -m cluster=mila resources=gpu algorithms/latent=umap data=swissroll
```

## What belongs here

This is a **public** repo. Only core infrastructure goes here:
- New LatentModules, metrics, callbacks, data modules
- Bug fixes, performance improvements, refactoring
- Generic config groups (algorithms, data, metrics, callbacks, cluster, resources)

Hydra instantiation configs (algorithm, data, metric, callback YAMLs) belong here — they're part of the core and CI depends on them.

**Do NOT push** experiment configs (sweep definitions), analysis scripts, data prep scripts, or project-specific sweeps. Those belong in the downstream repo that consumes manylatents (expaper repos, practitioner repos, etc.) — each has its own `experiments/configs/manylatents/experiment/` directory and a local manylatents pin.

## Releases

When tagging a release:
1. **Bump `version` in `pyproject.toml` first** — PyPI rejects uploads if the version already exists.
2. Run `uv run pytest tests/ -x -q` to verify.
3. Commit the version bump, push to main, then create the tag/release.
4. Watch the Publish to PyPI workflow with `gh run watch`.

## Pre-push checklist

**CI must pass before pushing to main.** Run:

```bash
uv run pytest tests/ -x -q && uv run pytest manylatents/callbacks/tests/ -x -q
```

If `gh` CLI is available, run CI locally **before pushing** to catch runner-specific failures early:

```bash
gh act -W .github/workflows/ci.yml   # run CI workflow locally via act
```

If `act` is not installed, at minimum verify checks immediately after push and fix before merging:

```bash
gh run watch   # watch the CI run after pushing
```

If CI fails after pushing, fix immediately — do not leave main broken.

## Running Experiments

**Always run experiment submissions as background tasks.** SLURM submissions and multirun sweeps can take time to dispatch — use `run_in_background: true` on the Bash tool so the conversation isn't blocked waiting. This applies to any `uv run python -m manylatents.main` invocation that submits to a cluster or runs a sweep.

## Config Discovery

**Don't hardcode config names** — discover them:

```bash
ls manylatents/configs/algorithms/latent/   # available LatentModule configs
ls manylatents/configs/algorithms/lightning/ # available LightningModule configs
ls manylatents/configs/data/                # datasets
ls manylatents/configs/metrics/             # all metrics (flat, each has at: field)
ls manylatents/configs/callbacks/embedding/ # callbacks
ls manylatents/configs/cluster/             # cluster profiles
```

For registered metrics: `uv run python -c "from manylatents.metrics import list_metrics; print(list_metrics())"`

## Adding New Components


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [latent-reasoning-works/manylatents](https://github.com/latent-reasoning-works/manylatents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-27 -->
