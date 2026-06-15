---
trigger: always_on
description: This file provides guidance to Coding Agent when working with code in this repository.
---

This file provides guidance to Coding Agent when working with code in this repository.

## What is openinfer

Pure Rust + CUDA LLM inference engine (~83K Rust, ~11K CUDA). No PyTorch, no frameworks. OpenAI-compatible `/v1/completions` API.

**Supported models:**

Every model line is behind a cargo feature; only `qwen3-4b` is a default feature, so the stock build is pure Rust + CUDA with no Python.

| Model | Crate | Feature flag | Architecture |
|-------|-------|-------------|-------------|
| Qwen3-4B / 8B | `openinfer-qwen3-4b` | `qwen3-4b` (default) | Full attention, TP support |
| Qwen3.5-4B | `openinfer-qwen35-4b` | `--features qwen35-4b` (needs build-time Python + Triton) | 24 linear + 8 full attention |
| DeepSeek-V4 | `openinfer-deepseek-v4` | `--features deepseek-v4` | MoE + compressor + indexer, 8-GPU |
| DeepSeek-V2-Lite | `openinfer-deepseek-v2-lite` | `--features deepseek-v2-lite` | MoE + EP, 2-GPU |
| Kimi-K2 | `openinfer-kimi-k2` | `--features kimi-k2` | MLA + MoE + Marlin INT4, 8-GPU EP |

## Build & Run

**Always use `--release`** — debug builds are extremely slow for GPU/CUDA and will timeout.

```bash
# Qwen3 (default feature, no Python anywhere in the build)
cargo run --release -- --model-path models/Qwen3-4B

# Feature-gated models
cargo run --release --features qwen35-4b -- --model-path models/Qwen3.5-4B
cargo run --release --features kimi-k2 -- --model-path models/Kimi-K2
cargo run --release --features deepseek-v4 -- --model-path models/DeepSeek-V4
```

**Key env vars:**
- `OPENINFER_CUDA_SM` — GPU SM target override when `nvidia-smi` unavailable (e.g. `120` or `120,80`)
- `OPENINFER_TRITON_PYTHON` — Python with Triton for `qwen35-4b` build-time AOT kernel generation (falls back to `.venv/bin/python`, then `python3`)
- `OPENINFER_TEST_MODEL_PATH` — override test model path (default: `models/Qwen3-4B`)
- `OPENINFER_BUILD_TIMING=1` — print per-phase build timings (nvcc, Triton AOT, etc.)
- `OPENINFER_NVCC_JOBS` — override parallel nvcc job count

## Tests

```bash
# Unit tests (~9s)
cargo test --release --workspace --lib

# Accuracy and integration tests — require GPU + model weights
OPENINFER_TEST_MODEL_PATH=models/Qwen3-4B cargo test --release -p openinfer-qwen3-4b --test hf_golden_gate
OPENINFER_TEST_MODEL_PATH=models/Qwen3.5-4B cargo test --release -p openinfer-qwen35-4b --features qwen35-4b --test hf_golden_gate
OPENINFER_TEST_MODEL_PATH=models/Qwen3.5-4B cargo test --release -p openinfer-qwen35-4b --features qwen35-4b --test e2e_scheduler

# Single test
cargo test --release embedding_variants -- --nocapture
```

Qwen accuracy gates compare logits against stored HF golden fixtures. Qwen3.5 exact-text JSON baselines are retired; keep `e2e_scheduler` for scheduler liveness and request-flow coverage.

## Architecture

```
HTTP Request → vLLM frontend → EngineHandle → per-model scheduler/executor → TokenEvent
                                               │
              ┌──────────────┬─────────────────┼─────────────────┬──────────────┐
              │              │                 │                 │              │
       openinfer-     openinfer-      openinfer-       openinfer-    openinfer-
       qwen3-4b      qwen35-4b      deepseek-v4     deepseek-v2-   kimi-k2
     (full attn)   (linear+full)   (MoE+indexer)    lite (MoE+EP)  (MLA+MoE)
              │              │                 │                 │              │
              └──────────────┴─────────────────┼─────────────────┴──────────────┘
                                               │
                         openinfer-core runtime + openinfer-kernels
                                               │
                              ┌────────────────┼────────────────┐
                              │                │                │
                      CUDA / cuBLAS    Triton AOT      FlashInfer
                                                    (sampling, attention,
                                                     norm, MLA decode)
```

**Key abstractions:**

- **`openinfer-core::engine`** — shared request/event contract (`EngineHandle`, `GenerateRequest`, `TokenEvent`) used by the server and model crates.
- **Per-model crates** — each model owns config, weights, prefill/decode execution, scheduler, tests, and benches.
- **`openinfer-core::ops`** — shared GPU operator wrappers used by model crates.
- **`openinfer-kernels`** — tensor/FFI/kernel build owner for CUDA, cuBLAS, FlashInfer, and Triton AOT. Model-specific kernels live in feature-gated submodules (`kimi_k2`, `deepseek_v4`).
- **`openinfer-comm`** — EP all-to-all communication (GDR, NCCL, IB verbs). Requires CUDA + RDMA hardware to compile.
- **CUDA Graph** — decode path captured inside model executors with pre-allocated buffers to preserve pointer stability.
- **KV state** — model schedulers own request state; shared paged-KV primitives live in `openinfer-core`.

**Build system**: the virtual workspace root has no package build script. `openinfer-kernels/build.rs` owns CUDA/Triton compilation:
1. Compiles `openinfer-kernels/csrc/*.cu` with nvcc (auto-detects GPU SM targets)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [openinfer-project/openinfer](https://github.com/openinfer-project/openinfer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
