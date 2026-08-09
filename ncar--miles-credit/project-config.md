---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## About

CREDIT (Community Research Earth Digital Intelligence Twin) is NSF NCAR's open-source platform for training
and deploying AI atmospheric prediction models (PyTorch). It targets both single-GPU laptops/workstations
and multi-node HPC (NCAR Casper/Derecho) via PBS.

## Commands

```bash
# Install (editable, no-deps into a pre-built shared conda env on Casper/Derecho)
pip install -e . --no-deps

# Install with dev extras (ruff, pytest, docs, jupyter) — general/non-NCAR envs
pip install -e ".[develop]"

# Lint & format (also runs via pre-commit)
ruff check --fix
ruff format

# Run the full test suite
pytest

# Run a single test file / test
pytest tests/test_models.py
pytest tests/test_models.py::test_crossformer_forward

# Coverage (matches CI)
pytest --cov=credit --cov-branch --cov-report=xml
```

Notes on testing:
- `tests/manual/` is excluded from collection (`norecursedirs` in `pyproject.toml`) — it holds
  multi-GPU/MPI/HPC tests that can't run on CI runners (see `tests/manual/gen2_parallelism/README.md`).
  Run these manually on Casper/Derecho via the `.pbs` scripts there.
- `tests/CI_CIRRUS/` is a separate self-hosted-runner GPU smoke test (real GFS init + rollout), not part
  of the standard `pytest` run.
- CI runs `pytest` on Python 3.11/3.12/3.13 with CPU-only torch (`ubuntu-latest`), so don't assume
  a GPU is present when writing tests.

## The `credit` CLI

Single entrypoint (`credit/cli/`, registered as `credit = credit.cli:main`): `init`, `check`, `preprocess`,
`train`, `rollout`, `rollout-ensemble` (deprecated), `realtime`, `submit`, `plot`, `metrics`, `convert`, `ask`.

`credit check -c config.yml [--deep] [--strict] [--json]` (`credit/cli/_check.py`) statically validates a gen2
config: resolves every registry key, binds each block's `args` against its real constructor signature,
cross-checks the channel layout and model geometry, verifies the BaseLoss target-twin postblock chain, and
existence-checks every path. Exits 1 on errors. When adding a registry entry, config key, or structural
constraint, add the matching check there — it is the fastest way for users to find the mistake.

`credit submit --cluster {casper,derecho} -c config.yml --gpus N [--nodes N] [--dry-run]` generates and
optionally submits a PBS batch script (`credit/pbs.py`), auto-chains multiple jobs via `afterok` dependencies
based on `trainer.epochs / trainer.num_epoch`, and resolves the PBS account as
CLI flag → config `pbs:` block → `$PBS_ACCOUNT` env var → default. Always sanity-check with `--dry-run` first.

There are also legacy standalone entry points (`credit_train_gen1`, `credit_rollout_realtime`,
`credit_gfs_init`, etc., see `pyproject.toml` `[project.scripts]`) kept for backward compatibility —
prefer the unified `credit` CLI for new work.

## Architecture

### gen1 vs gen2 — the central fork in this codebase

CREDIT has two parallel generations of the data/training pipeline living side by side. Know which one
a file belongs to before editing it:

|              | gen1 (legacy)                          | gen2 (current)                                    |
|--------------|------------------------------------------|----------------------------------------------------|
| Trainer      | `credit/trainers/trainerERA5gen1.py` (`trainer.type: era5`) | `credit/trainers/trainer_gen2.py` (`trainer.type: era5-gen2`) |
| Dataset      | `credit/datasets/gen_1/` — flat schema  | `credit/datasets/gen_2/` — nested `data.source.<name>` schema, typed channels (`prognostic`/`dynamic_forcing`/`static`/`diagnostic`) via `gen_2/schema.py` |
| `forecast_len` semantics | `0` = single step | `1` = single step (gen2 is 1-indexed) |
| Pre/post-processing | conservation fixers called directly (`credit/postblock/gen1.py`) | explicit `build_preblocks`/`apply_preblocks` and `build_postblocks`/`apply_postblocks` pipeline stages |
| Config location | `config/gen_1/` | `config/gen_2/` |

Both trainers inherit `credit/trainers/base_trainer.py`. `credit convert` migrates a v1/gen1 config to gen2
(bumps `forecast_len`/`valid_forecast_len` by 1, retargets `trainer.type`, prompts for EMA/TensorBoard/PBS
settings). Prefer gen2 for new work; gen1 is kept for reference/reproducibility (see `config/gen_1/arXiv_2024/`).

### Registry pattern (extensibility)

Five subsystems — models, trainers, datasets, preblocks, postblocks, losses — are selected purely by a
string key in the config (`model.type`, `trainer.type`, `data.source.<name>.dataset_type`, etc.), resolved
through per-package `_REGISTRY` dicts with lazy imports:
- `credit/models/__init__.py` → `load_model(conf)`, keys like `crossformer`, `wxformer`, `unet`, `fuxi`, `swin`, `graph`
- `credit/trainers/__init__.py` → keys like `era5-gen1`/`era5`, `era5-gen2`/`gen2`, `era5-diffusion`, `era5-ensemble`, `ic-opt`
- `credit/datasets/`, `credit/preblock/`, `credit/postblock/`, `credit/losses/` follow the same `register_*` decorator pattern

`credit/registry.py` (`load_custom_objects`) is the meta-layer: a config's `custom_objects:` block lets users

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NCAR/miles-credit](https://github.com/NCAR/miles-credit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
