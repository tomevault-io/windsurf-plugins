---
trigger: always_on
description: This file provides guidance to Coding Agent when working with code in this repository.
---

This file provides guidance to Coding Agent when working with code in this repository.

## What is PegaInfer

Pure Rust + CUDA LLM inference engine. No PyTorch, no frameworks. OpenAI-compatible `/v1/completions` API.

**Supported models:**

Every model line is behind a cargo feature; only `qwen3` is a default feature, so the stock build is pure Rust + CUDA with no Python.

| Model | Crate | Feature flag | Architecture |
|-------|-------|-------------|-------------|
| Qwen3-4B / 8B | `pegainfer-qwen3` | `qwen3` (default) | Full attention, TP support |
| Qwen3.5-4B / 9B / 27B | `pegainfer-qwen35` | `--features qwen35` (needs build-time Python + Triton) | Hybrid Gated DeltaNet + full attention |
| DeepSeek-V2-Lite | `pegainfer-deepseek-v2-lite` | `--features deepseek-v2-lite` | MoE + EP, 2-GPU |
| Gemma 4 | `pegainfer-gemma4` | `--features gemma4` | Registration only — engine not yet available |
| Kimi-K2 | `pegainfer-kimi-k2` | `--features kimi-k2` | MLA + MoE + Marlin INT4, 8-GPU EP |
| GLM5.2 | `pegainfer-glm52` | `--features glm52` | MLA + MoE + FP8, 8-GPU EP (bring-up) |

## Build & Run

**Always use `--release`** — debug builds are extremely slow for GPU/CUDA and will timeout.

When developing with Docker, use `docker/Dockerfile.dev` and `docker/dev.sh` as described in `docker/README.md`.

```bash
# Qwen3 (default feature, no Python anywhere in the build)
cargo run --release -- --model-path models/Qwen3-4B

# Feature-gated models
cargo run --release --features qwen35 -- --model-path models/Qwen3.5-4B
cargo run --release --features kimi-k2 -- --model-path models/Kimi-K2
cargo run --release --features deepseek-v2-lite -- --model-path models/DeepSeek-V2-Lite
cargo run --release --features glm52 -- --model-path models/GLM5.2
```

**Key env vars:**
- `PEGAINFER_CUDA_SM` — GPU SM target override when `nvidia-smi` unavailable (e.g. `120` or `120,80`)
- `PEGAINFER_TRITON_PYTHON` — Python with Triton for `qwen35` build-time AOT kernel generation (falls back to `.venv/bin/python`, then `python3`, then `python`)
- `PEGAINFER_TILELANG_PYTHON` — Python with TileLang for the `glm52` sparse-MLA build-time AOT (sm_90a targets only)
- `PEGAINFER_NCCL_ROOT` — NCCL root (>= 2.30.4) for DeepEP shim (`moe` feature)
- `PEGAINFER_FLASHINFER_INCLUDE` — FlashInfer include dir override
- `PEGAINFER_TEST_MODEL_PATH` — override test model path (default: `models/Qwen3-4B`)
- `PEGAINFER_BUILD_TIMING=1` — print per-phase build timings (nvcc, Triton AOT, etc.)
- `PEGAINFER_NVCC_JOBS` — override parallel nvcc job count
- `GLM52_DECODE_SLOTS` / `GLM52_MTP_DRAFTS` — glm52 runtime profile: decode slots per rank (default 8, ceiling 32) and MTP draft span (default 5); `slots x (1+drafts)` must fit the 96-row step (validated at launch; MTP only). Throughput ceiling profile: `32` / `2`.

## Tests

```bash
# Unit tests (~9s)
cargo test --release --workspace --lib

# Accuracy and integration tests — require GPU + model weights
cargo test --release -p pegainfer-qwen3 --test hf_golden_gate
PEGAINFER_TEST_MODEL_PATH=models/Qwen3.5-4B cargo test --release -p pegainfer-qwen35 --features qwen35 --test hf_golden_gate
PEGAINFER_TEST_MODEL_PATH=models/Qwen3.5-4B cargo test --release -p pegainfer-qwen35 --features qwen35 --test e2e_scheduler

# Single test (filter by name)
cargo test --release --workspace --lib prefix_cache -- --nocapture
```

Qwen accuracy gates compare logits against stored HF golden fixtures. Qwen3.5 exact-text JSON baselines are retired; keep `e2e_scheduler` for scheduler liveness and request-flow coverage.

## Architecture

```
HTTP Request → vLLM frontend → EngineHandle → per-model scheduler/executor → TokenEvent
                                               │
              ┌──────────┬─────────────┬───────┼───────────┬──────────┐
              │          │             │       │           │          │
        pegainfer-  pegainfer-   pegainfer-  pegainfer-  pegainfer-  ...
        qwen3       qwen35       dsv2-lite   kimi-k2     glm52
      (full attn) (linear+full) (MoE+EP)   (MLA+MoE)  (MLA+MoE+FP8)
              │          │             │       │           │          │
              └──────────┴─────────────┴───────┼───────────┴──────────┘
                                               │
                          pegainfer-core runtime + pegainfer-kernels
                                               │
                               ┌───────────────┼───────────────┐
                               │               │               │
                       CUDA / cuBLAS    Triton AOT      FlashInfer
                                                    (sampling, attention,
                                                     norm, MLA decode)
```

**Key abstractions:**

- **`pegainfer-frontend`** — the serving frontend: the engine request/event contract (`pegainfer_frontend::engine` — `EngineHandle`, `GenerateRequest`, `TokenEvent`) plus the protocol stacks on top of it (`vllm` module today, `dynamo` planned) and the `ModelLine` dispatch trait. Model crates implement against the contract; the server binary does pure dispatch.
- **Per-model crates** — each model owns config, weights, prefill/decode execution, scheduler, tests, and benches.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pegainfer-project/pegainfer](https://github.com/pegainfer-project/pegainfer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-10 -->
