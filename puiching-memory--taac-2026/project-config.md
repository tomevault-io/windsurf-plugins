---
trigger: always_on
description: This repository is the TAAC 2026 experiment workspace. It combines reusable PCVR
---

# AGENTS.md

This repository is the TAAC 2026 experiment workspace. It combines reusable PCVR
training, evaluation, inference, online bundle packaging, directory-based
experiment packages, tests, and documentation.

Use this file as the operating guide for AI coding agents working in this repo.

## Project Shape

- `src/taac2026/` contains shared framework code.
- `experiments/` contains loadable experiment packages.
- `docs/` contains Zensical documentation source.
- `tests/` contains unit, contract, integration, benchmark, and GPU tests.
- `run.sh` is the local and bundle-compatible command entrypoint.
- `pyproject.toml` and `uv.lock` define the Python environment.

Keep experiment packages thin. Shared training, evaluation, inference, data
pipeline, checkpoint, optimizer, accelerator, and packaging behavior belongs
under `src/taac2026/`, not copied into `experiments/`.

## Read First

Before changing non-trivial code, read the relevant docs:

- `README.md` for project overview and common commands.
- `docs/architecture.md` for layering and module responsibilities.
- `docs/guide/testing.md` for validation commands.
- `docs/guide/online-training-bundle.md` when touching packaging or platform
  bundle behavior.
- `docs/experiments/index.md` and the specific experiment doc when changing an
  experiment package.

## Environment

This is a Linux-only project managed by `uv`.

Common setup:

```bash
uv sync --locked --extra dev --extra cuda126
```

CPU-only unit work may only need:

```bash
uv sync --locked --extra dev
```

Do not hand-edit `uv.lock` unless dependency changes require it. Use `uv`
commands to update lock state.

## Common Commands

Run unit tests:

```bash
uv run pytest tests/unit -q
```

Run CPU-safe PR-style checks:

```bash
uv run pytest -m "(unit or contract or integration or benchmark_cpu) and not gpu and not benchmark_gpu" -q
```

Run experiment package contract tests:

```bash
uv run pytest tests/contract/experiments/test_packages.py -q
uv run pytest tests/contract/experiments/test_runtime_contract_matrix.py -q
```

Run packaging/bootstrap tests:

```bash
uv run pytest tests/integration/application/packaging -q
uv run pytest tests/unit/application/bootstrap tests/integration/application/bootstrap -q
```

Run lint:

```bash
uv run ruff check .
```

Build docs strictly:

```bash
uv run zensical build --strict
```

Run a small CPU smoke train:

```bash
bash run.sh train \
  --experiment experiments/baseline \
  --run-dir outputs/baseline_smoke \
  --device cpu \
  --num_workers 0 \
  --batch_size 8 \
  --max_steps 1
```

## Layering Rules

Respect the dependency direction.

- `domain/`: pure contracts, config, schema, requests, metrics, sidecar, and
  model input contracts.
- `application/`: train/eval/infer/packaging/bootstrap workflows and CLI
  orchestration.
- `infrastructure/`: data, IO, runtime, modeling, optimization, accelerators,
  and platform utilities.
- `experiments/`: experiment name, defaults, model class, experiment-private
  layers, and experiment-private hooks.

Do not put CLI parsing, environment probing, or filesystem behavior in
`domain/`.

Do not put experiment selection policy into `infrastructure/`.

Do not duplicate shared runtime logic inside experiment packages.

## Experiment Package Rules

A normal experiment package should usually look like:

```text
experiments/my_experiment/
├── __init__.py
└── model.py
```

Optional private model code can go in `layers.py`.

`__init__.py` should normally expose `EXPERIMENT` using
`create_pcvr_experiment()`.

`model.py` must satisfy the PCVR runtime contract:

- `forward(inputs)` returns logits shaped `(B,)`.
- `predict(inputs)` returns `(logits, embeddings)`.
- Model construction must work from checkpoint sidecar config and schema.
- Sparse and dense parameters should be groupable by the shared optimizer logic.

Prefer shared imports from `taac2026.api` and
`taac2026.infrastructure.modeling` over local copies.

## Checkpoint And Sidecar Contract

A valid trained checkpoint directory must include:

```text
model.safetensors
schema.json
train_config.json
```

Do not assume `model.safetensors` alone is enough. Evaluation and inference
reconstruct schema, input contracts, NS config, model defaults, runtime
execution settings, and hooks from sidecar files.

Be careful when changing:

- `PCVRModelConfig`
- `PCVRTrainConfig`
- `PCVRNSConfig`
- `ModelInput`
- schema serialization
- checkpoint directory naming
- sidecar read/write behavior

## Data And Local Runs

Local PCVR smoke runs use the Hugging Face sample dataset through the repository
data path conventions. Do not add or require `--dataset-path` for normal local
PCVR smoke tests unless the relevant docs and code explicitly support it.

Online bundles receive real data paths from the platform. Keep local assumptions
separate from platform bundle behavior.

## Packaging Rules

When changing bundle behavior, verify both tests and actual zip contents.

Training bundle command:

```bash
uv run taac-package-train \
  --experiment experiments/baseline \
  --output-dir outputs/bundles/baseline_training \
  --json
```

Inference bundle command:

```bash
uv run taac-package-infer \
  --experiment experiments/baseline \

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Puiching-Memory/TAAC_2026](https://github.com/Puiching-Memory/TAAC_2026) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
