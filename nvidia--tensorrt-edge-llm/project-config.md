---
trigger: always_on
description: TensorRT Edge-LLM: NVIDIA C++/CUDA/Python inference runtime for deploying LLMs and VLMs on edge devices (Jetson Orin, Thor, DRIVE platforms).
---

# AGENTS.md

TensorRT Edge-LLM: NVIDIA C++/CUDA/Python inference runtime for deploying LLMs and VLMs on edge devices (Jetson Orin, Thor, DRIVE platforms).

> If an `AGENTS.local.md` file exists alongside this file, read and respect it — it contains developer-specific overrides that supplement this shared guidance.

## Rules (Read First)

**CRITICAL (YOU MUST):**
- Read and follow `CODING_GUIDELINES.md` for ALL code changes (C++ and Python)
- NVIDIA copyright header on ALL new files (update year on modified files) — see `LICENSE_HEADER` for the SPDX template (pre-commit `insert-license` hook auto-injects it for `.py`, `.cpp`, `.cu`, `.cuh`, `.h`, `.hpp` files)
- `git commit -s` (DCO sign-off required). Never attribute AI tools in sign-off line. Always rely on `git` to do the sign off instead of directly adding sign off in commit message.
- Do not add co-authors to the git commit message unless explicitly instructed to do so by the user.
- `pre-commit` hooks run on commit — if files are modified by hooks, re-stage and commit again
- PR title format: Conventional Commits style (e.g., `feat: Add Qwen3 support`, `fix #700: Memory leak in runtime`)
- Set `TRT_PACKAGE_DIR` for all C++ builds; set `LLM_SDK_DIR` for all Python tests
- Set `LD_LIBRARY_PATH` before running any built binary: `export LD_LIBRARY_PATH=$TRT_PACKAGE_DIR/lib:$LD_LIBRARY_PATH`
- Git submodules must be initialized: `git submodule update --init` (googletest, nlohmann/json, NVTX)

## Common Commands

| Task | Command |
|------|---------|
| Build (minimal) | `mkdir -p build && cd build && cmake .. -DTRT_PACKAGE_DIR=$TRT_PACKAGE_DIR && make -j$(nproc)` |
| Build (with unit tests) | `cmake .. -DTRT_PACKAGE_DIR=$TRT_PACKAGE_DIR -DBUILD_UNIT_TESTS=ON && make -j$(nproc)` |
| Build (cross-compile AArch64) | `cmake .. -DTRT_PACKAGE_DIR=$TRT_PACKAGE_DIR -DAARCH64_BUILD=ON && make -j$(nproc)` |
| Build (NVTX profiling) | `cmake .. -DTRT_PACKAGE_DIR=$TRT_PACKAGE_DIR -DENABLE_NVTX_PROFILING=ON && make -j$(nproc)` |
| C++ unit tests (all) | `./build/unitTest` |
| C++ unit tests (filter) | `./build/unitTest --gtest_filter="LoggerTest.*"` |
| Python package (install) | `pip install -r requirements.txt && python -m build --wheel --outdir dist . && pip install dist/*.whl` |
| Python test suite | `pytest --priority=l0_pipeline_a30 -v` |
| Single Python test | `pytest tests/defs/test_model_export.py -v` |
| Python unit tests | `pytest tests/python-unittests/ -v` |
| Pre-commit (all files) | `pre-commit run --all-files` |
| Pre-commit (specific file) | `pre-commit run --files cpp/runtime/llmInferenceRuntime.cpp` |
| Format C++ manually | `git-clang-format --style file` |
| Code coverage | `./scripts/run_coverage.sh --trt-package-dir $TRT_PACKAGE_DIR` |

For the full end-to-end pipeline (quantize → ONNX export → engine build → inference), installation instructions, and build options, see `docs/source/developer_guide/getting-started/`.

## Architecture

For detailed software design, see `docs/source/developer_guide/software-design/`.

The pipeline is: `HuggingFace Model → Python Export (quantize + ONNX) → C++ Engine Builder (TRT engine) → C++ Runtime (inference)`.

**C++ Runtime (`cpp/`)** has two mutually exclusive inference paths, both via `handleRequest()`:
- `LLMInferenceRuntime` — standard inference (single engine + tokenizer + optional multimodal)
- `LLMInferenceSpecDecodeRuntime` — EAGLE speculative decoding (base + draft engines)

**C++ sub-packages:** `common/` (tensor, logging, utils), `kernels/` (FMHA/RoPE/MoE/Mamba/EAGLE), `plugins/` (TRT custom plugins), `builder/` (ONNX→TRT), `tokenizer/`, `multimodal/`, `profiling/`, `sampler/`.

**Python package (`tensorrt_edgellm/`)** mirrors HuggingFace model interfaces with quantization-aware rewrites. See `docs/source/developer_guide/software-design/python-export-pipeline.md`.

**Experimental LLM Loader (`experimental/llm_loader/`)** is a next-gen checkpoint-based model loader that implements LLM architectures from scratch using ONNX builtin + custom ops (the only format EdgeLLM's compiler accepts). Instead of tracing HuggingFace FX graphs (which are unstable), it reads the stable HF checkpoint weights directly.
- `model.py` — `AutoModel.from_pretrained()` factory with registry-based dispatch
- `config.py` — `ModelConfig`/`QuantConfig` for parsing HF `config.json`
- `checkpoint/loader.py` — Safetensors weight loading; `repacking.py` — weight repacking
- `onnx/export.py` — Export via `torch.onnx.export(dynamo=True)`; `onnx_custom_schemas.py` — custom op definitions; `dynamo_translations.py` — custom translation rules
- `models/` — Per-architecture implementations: `default/` (standard decoder + Mamba hybrid), `nemotron_h/` (hybrid Mamba2), `qwen3_moe/` (sparse MoE)
- `models/ops.py` — Shared custom operations; `models/linear.py` — Shared linear layer implementations
- Supported quant formats: `fp16`, `fp8`, `nvfp4`, `int4_awq`, `int4_awq_modelopt`, `int4_gptq`, `int8_sq`, `mixed_precision`

### CLI Entry Points (from `pyproject.toml`)

| Command | Script |
|---------|--------|
| `tensorrt-edgellm-quantize-llm` | `tensorrt_edgellm.scripts.quantize_llm:main` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NVIDIA/TensorRT-Edge-LLM](https://github.com/NVIDIA/TensorRT-Edge-LLM) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
