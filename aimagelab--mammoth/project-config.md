---
trigger: always_on
description: Terminal-agent manual for `mammoth`.
---

# AGENTS.md

Terminal-agent manual for `mammoth`.

This file is the canonical guide for coding agents in this repository.

## 1) Scope and priorities

- Scope: entire repo.
- Language: Python (`>=3.10`).
- Entrypoint: `main.py`.
- Highest-priority area: `models/`.
- Then: `datasets/`, then `backbone/`.
- Primary objective: preserve continual-learning correctness and reproducibility.
- Secondary objective: keep diffs small and targeted.

### Quick do/don't (models)

Do:

- edit `models/` first for method behavior changes,
- keep `observe` signature and return contract stable,
- run model-focused tests before broad suites.

Don't:

- bypass `meta_*` wrappers in training flow,
- change parser/config behavior without updating related tests,
- refactor unrelated files in the same change.

### Quick do/don't (datasets)

Do:

- keep train loader output as `(augmented, label, non_augmented)`,
- use `store_masked_loaders(...)` unless a setting truly requires custom splitting,
- validate task/mask behavior after any split/config change.

Don't:

- manually bypass `MammothDatasetWrapper` behavior for standard continual datasets,
- break `custom_task_order`/`custom_class_order` and permutation interactions,
- change dataset defaults/config semantics without parser tests.

### Quick do/don't (backbones)

Do:

- register new backbones with `@register_backbone`,
- keep constructor signature parser-friendly (dynamic args are inferred),
- preserve `forward(..., returnt=...)` compatibility used by existing models.

Don't:

- introduce required ctor args without exposing parseable arguments,
- change feature/logit output conventions without auditing dependent models,
- rely on only one `returnt` variant if models need `out/features/both/full`.

## 2) Repo intent

Mammoth is a continual-learning framework used to:

- run many CL methods across many datasets,
- support multiple settings (`class-il`, `domain-il`, `task-il`, `general-continual`, `cssl`, `biased-class-il`),
- make it easy to add models, datasets, and backbones.

## 3) Environment and canonical commands

Prefer `uv`.

Setup:

1. `pip install uv`
2. `uv sync --group dev`
3. Run everything with `uv run <command>`

Sanity commands:

- `uv run python main.py --help`
- `uv run pip install -e .`
- `uv run python -m build`

Tests:

- Full suite: `uv run pytest`
- Fast smoke: `uv run pytest tests/test_import.py tests/test_basic_functionality.py`
- Single file: `uv run pytest tests/test_er_example.py`
- Pattern: `uv run pytest -k "checkpoint and not wandb"`

Pytest options used in this repo:

- `--force_device=<id>` sets `MAMMOTH_DEVICE` and `CUDA_VISIBLE_DEVICES`.
- `--include_dataset_reload` enables destructive/slow dataset re-download tests.

Formatting/linting:

- `uv run autopep8 --recursive --in-place --aggressive --max-line-length=200 --ignore=E402 .`
- `uv run ruff check .`
- `uv run mypy .`

Docs:

- `cd docs && uv run ../utils/args.py && uv run sphinx-build -j auto . ../_build`
- or `cd docs && make html`

## 4) Where to edit

- `main.py`: parser pipeline, config merge, initialization.
- `models/`: methods.
- `models/utils/continual_model.py`: base model contract and wrappers.
- `models/utils/future_model.py`: required for `--eval_future`.
- `models/config/*.yaml`: model `default`/`best` configs.
- `datasets/`: dataset definitions.
- `datasets/utils/continual_dataset.py`: dataset contract + `store_masked_loaders`.
- `datasets/configs/<dataset>/*.yaml`: dataset configs.
- `backbone/`: backbone implementations + registration.
- `utils/args.py`: dynamic parser behavior.
- `utils/training.py`: training loop and `meta_*` calls.
- `utils/evaluate.py`: evaluation semantics and compatibility consequences.
- `tests/`: validation.

## 5) Runtime flow (debug map)

Core execution path:

1. `parse_args()` in `main.py` builds parser in stages.
2. Model parser and config files are loaded.
3. Dynamic dataset/backbone args are injected.
4. `initialize()` builds dataset -> backbone -> model.
5. `train()` in `utils/training.py` runs task loop using:
   - `model.meta_begin_task(dataset)`
   - `model.meta_observe(...)`
   - `model.meta_end_task(dataset)`

When behavior looks wrong, inspect `meta_*` wrapper behavior first.

## 6) Models (most important)

### Discovery

- `models/__init__.py` supports both:
  - explicit registry (`register_model`, `REGISTERED_MODELS`), and
  - legacy autodiscovery from `models/*.py` via `ContinualModel` subclass detection.
- Name matching tolerates `_` and `-` differences.

### Required model contract

Every model must:

- inherit `models.utils.continual_model.ContinualModel`,
- define `NAME` and `COMPATIBILITY`,
- implement `observe(self, inputs, labels, not_aug_inputs, epoch=None, ...)`.

`observe` can return either:

- scalar loss, or
- dict with at least `loss`.

`forward` nuance:

- If `'class-il'` and `'general-continual'` are both missing from `COMPATIBILITY`, evaluation calls `model(inputs, task_id)`.
- In that case, ensure `forward` supports a task label argument.

### Wrapper semantics

`meta_observe` handles critical framework behavior:

- removes unlabeled samples (`label == -1`) for models without `'cssl'` compatibility,
- filters extra kwargs to match `observe` signature,
- asserts required observe args are present,
- handles wandb autologging.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aimagelab/mammoth](https://github.com/aimagelab/mammoth) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
