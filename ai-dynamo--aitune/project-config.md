---
trigger: always_on
description: This file provides guidance to AI coding agents when working with code in this repository.
---

# AGENTS.md

This file provides guidance to AI coding agents when working with code in this repository.

## Development Commands

```bash
make install-dev          # editable install with all dev deps (uses --extra-index-url https://pypi.nvidia.com)
make lint                 # pre-commit --all-files + pytype
make test                 # pytest (unit tests + doctests)
make coverage             # coverage report → htmlcov/
make docs-serve           # mkdocs at http://localhost:8000

pytest tests/unit/path/to/test_file.py::test_name -v   # single test
pytest -k "keyword" -v                                  # filter by keyword
pytest --no-header -rN tests/unit/ -x                  # fail fast
```

Test paths are `aitune/` and `tests/unit/` (doctests are enabled across production code). Functional tests under `tests/functional/` require GPU + real backends.

Documentation source of truth: backend guides live under `docs/guides/backends/`, and example docs live in `examples/*/README.md` with `examples/README.md` as the catalog. Do not hand-maintain duplicate copies under `docs/examples/`.

**Environment variables — runtime:**
- `AITUNE_CACHE_DIR` — AOT cache directory (default: `~/.cache/aitune`)
- `AITUNE_JIT_CACHE_DIR` — JIT cache directory (default: `~/.cache/aitune.jit`)
- `AITUNE_CONSOLE_OUTPUT=1` — print tuning progress to stdout
- `AITUNE_NVTX_EVENTS=1` — enable NVTX profiling annotations
- `AITUNE_HARDWARE_METRICS=1` — collect GPU hardware metrics during tuning
- `AITUNE_HARDWARE_METRICS_PATH` — file path for hardware metrics output
- `AITUNE_TUNING_DATA_PATH` — file path for tuning telemetry output (telemetry is always collected)
- `AUTOWRAPT_BOOTSTRAP=aitune_enable_jit_tuning` — enable JIT mode without modifying source

**Environment variables — tests:**
- `AITUNE_TESTS_USE_DEVICE` — override device (defaults to `cuda:0` if available, else CPU)
- `AITUNE_TESTS_LOG_LEVEL=DEBUG` — verbose logging during test runs

## Architecture

AITune wraps PyTorch `nn.Module` subgraphs and compiles them with an optimized backend (TensorRT, Torch-TensorRT, TorchInductor, TorchAO, ONNXRuntime). There are two usage modes:

### Ahead-of-Time (AOT / Declarative) Mode

The user explicitly calls the API:

```python
modules_info = aitune.torch.inspect(model, dataset)  # find tunable submodules
model = aitune.torch.wrap(model, modules_info.get_modules())  # wrap selected modules
aitune.torch.tune(model, dataset, batch_sizes=[1, 2])  # compile with chosen backends
aitune.torch.save(model, "model.ait")  # persist compiled engines
model = aitune.torch.load(model, "model.ait")  # restore in production
```

### Just-In-Time (JIT) Mode

No code changes required. The module is monkey-patched at import time:

```python
import aitune.torch.jit.enable  # must be the first import
# or via env: AUTOWRAPT_BOOTSTRAP=aitune_enable_jit_tuning python script.py
```

JIT sub-modes (`aitune.torch.jit_config.mode`, type `JITMode`):
- `TUNE_EAGER` *(default)* — tunes automatically once each module has seen `min_samples` runs
- `TUNE_DEFERRED` — collects samples but waits for an explicit `tune.deferred()` call; use for diffusion pipelines where modules run a variable number of times per step
- `INSPECT` — records execution metadata and emits an HTML report, never compiles

JIT also supports `detect_graph_breaks` (uses `torch._dynamo.explain()` to skip modules that would break compilation) and `max_depth_level` (skips modules deeper than N levels in the hierarchy).

### Key Modules

| Module | Role |
|---|---|
| `aitune/torch/tuning.py` | `tune()`, `save()`, `load()` — top-level AOT API |
| `aitune/torch/inspecting/` | `inspect()` / `wrap()` — traverses module hierarchy to find candidates |
| `aitune/torch/module/` | `Module` wrapper and `ModuleState` FSM (INITIAL → RECORDING → TUNED / PASSTHROUGH) |
| `aitune/torch/backend/` | One class per backend; all extend `Backend` base class in `backend.py` |
| `aitune/torch/tune_strategy/` | `OneBackendStrategy`, `FirstWinsStrategy`, `MaxThroughputStrategy` |
| `aitune/torch/jit/` | `Patcher`, `PatchedModule`, `enable.py` (autowrapt bootstrap), JIT config |
| `aitune/torch/checkpoint/` | `TorchCheckpoint` + `LocalTorchStorage` — gzip-compressed save/load, files named by SHA256 hash |
| `aitune/torch/dataloader.py` | `DataLoaderFactory`, `samples_generator()` — normalises heterogeneous dataset types |
| `aitune/torch/task/` | `profile()` + measuring strategies — profiling infrastructure used by `MaxThroughputStrategy` |
| `aitune/torch/config.py` | `AITuneConfig` singleton (`aitune.torch.config`) |
| `aitune/torch/jit/config.py` | `Config` dataclass singleton (`aitune.torch.jit_config`) |
| `aitune/utils/env_vars.py` | All env-var parsing in one place |
| `aitune/dynamo/` | `DynamoWorker` — serve tuned models as Dynamo endpoints (requires `aitune[dynamo]`) |

### Backend Implementations

Located in `aitune/torch/backend/`:
- `tensorrt/tensorrt_backend.py` — TensorRT with optional quantization (requires `use_dynamo` flag)
- `torch_tensorrt_jit_backend.py` / `torch_tensorrt_aot_backend.py`
- `torch_inductor_jit_backend.py` / `torch_inductor_aot_backend.py`
- `torchao_backend.py`
- `torch_eager.py` — pass-through baseline
- `onnx_runtime_backend.py`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ai-dynamo/aitune](https://github.com/ai-dynamo/aitune) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
