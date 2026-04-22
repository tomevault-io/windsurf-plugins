---
trigger: always_on
description: This file stores repo-specific priors for future agents. Keep it short, practical,
---

## StepTronOSS AGENTS

This file stores repo-specific priors for future agents. Keep it short, practical,
and biased toward things that save repeated exploration.

## 1. Working Rules

### Self-Improvement Loop

Do an improve pass for key tasks:

- new environment
- new task type
- new project area
- high risk / high cost work
- collaboration / handoff work

Improve pass:

1. Identify friction
2. Extract reusable priors
3. Write them down in:
   - `AGENTS.md` for repo-wide priors
   - `docs/` for process / runbook details

## 2. Repo Layout

- Core package: `steptronoss/`
  - core, model, data, exp, optimizer, generation, tokenizer, utils, checkpointing
- Experiments: `playground/`
- Tests: `tests/`
- Docs: `docs/`

### Utilities overview

- `steptronoss/utils/arguments.py`: config overrides from CLI
- `steptronoss/utils/comm_utils.py`: Redis rendezvous, queues, `LocalFuture` / `RemoteFuture`
- `steptronoss/utils/dist_utils.py`: broadcast / all-to-all helpers, packing helpers, balancing helpers
- `steptronoss/utils/general.py`: numeric helpers, list split/balance, RNG fork, retry, recursion helpers, git hash
- `steptronoss/utils/logger.py`: rank-aware logging and `StepWriter`
- `steptronoss/utils/metrics.py`: metrics system (`Metric`, `Avg`, `Percentage`, `Histogram`, `Text`, `GradNorm`, `GlobalMetrics`)
- `steptronoss/utils/optimizable.py`: `@optimizable(...)` and `set_optimization(...)`
- `steptronoss/utils/utils.py`: model unwrap, param norms, memory report, layer map, IO helpers, generic load
- `steptronoss/utils/weight_loader.py`: HF safetensors mapping / merge

## 3. Code Style

### Config style

- Config class fields should include a short triple-quoted docstring immediately after the attribute definition.
- Follow the `configurize` pattern:
  - class attrs declare sub-config types
  - instance `__init__` sets concrete values
  - use `Ref("..path")` for cross-node linkage
  - configs expose `build()` / `build_*`, `sanity_check()`, `to_dict()`
- Only `Ref(...)` the exact parameter needed, not whole config objects.

### Experiment style

- SFT experiments under `playground/sft/qwen3/*_sft_step3_data.py` typically follow:
  - `class Exp(BaseExp)`
  - `model_cfg` / `data_cfg` declared as class attrs
  - trainer / checkpoint / model fields adjusted in `__init__`
  - entrypoint is `if __name__ == "__main__": Exp().train()`

## 4. Setup Priors

- After `uv sync`, also install `redis-server`:
  - `apt install -y redis-server`

### DeepEP build

- Set:
  - `CUDA_HOME=/data/cuda/cuda-12.9/cuda`
  - `CUDACXX=$CUDA_HOME/bin/nvcc`
- Install:
  - `pip install -e /data/DeepEP --no-build-isolation`

### nv-grouped-gemm build

- Do not rely on random prebuilt wheels; ABI mismatch is common.
- Build with CUDA 12.9:
  - `CUDA_HOME=/data/cuda/cuda-12.9/cuda CUDACXX=/data/cuda/cuda-12.9/cuda/bin/nvcc <python> -m pip install -e <grouped_gemm_source> --no-build-isolation`
- Runtime constraints:
  - `batch_sizes` must be CPU-visible / `torch.int64`
  - inputs must be bf16 for `nv_grouped_gemm`

## 3. Code Style

### Config style

- Config class fields should include a short triple-quoted docstring immediately after the attribute definition.
- Follow the `configurize` pattern:
  - class attrs declare sub-config types
  - instance `__init__` sets concrete values
  - use `Ref("..path")` for cross-node linkage
  - configs expose `build()` / `build_*`, `sanity_check()`, `to_dict()`
- Only `Ref(...)` the exact parameter needed, not whole config objects.

### Experiment style

- SFT experiments under `playground/sft/qwen3/*_sft_step3_data.py` typically follow:
  - `class Exp(BaseExp)`
  - `model_cfg` / `data_cfg` declared as class attrs
  - trainer / checkpoint / model fields adjusted in `__init__`
  - entrypoint is `if __name__ == "__main__": Exp().train()`
- `playground/sft/qwen3/qwen3_sft_base.py` already provides `OneNodeResourceConfig` with `replica=1` and `gpu=8`, so derived SFT experiments default to single-node 8-GPU `torchrun` unless they override `resource_cfg`.
- Under `playground/data/sft`, keep raw source recipes and dataset configs distinct in naming:
  - `*_recipe*.py` for `DataRecipe` / source file lists only
  - `*_data_config*.py` for `CompliableDatasetsConfig`, `CompiledDatasetsConfig`, and `SFTDataConfig`
  - when adding tokenizer variants for the same source recipe, share a common base config and use thin tokenizer-specific subclasses instead of duplicating whole modules
  - for large unified SFT rebuilds, do not mutate source json/jsonl in place; materialize derived json under `/oss/...`, preserve `DataSourceFile.subsample_rate` semantics with sample seed `1234`, and do global shuffle with an external bucketized sort instead of loading everything into memory

## 4. Setup Priors

- After `uv sync`, also install `redis-server`:
  - `apt install -y redis-server`

### DeepEP build

- Set:
  - `CUDA_HOME=/data/cuda/cuda-12.9/cuda`
  - `CUDACXX=$CUDA_HOME/bin/nvcc`
- Install:
  - `pip install -e /data/DeepEP --no-build-isolation`

### nv-grouped-gemm build

- Do not rely on random prebuilt wheels; ABI mismatch is common.
- Build with CUDA 12.9:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [stepfun-ai/SteptronOss](https://github.com/stepfun-ai/SteptronOss) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
