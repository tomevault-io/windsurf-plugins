---
trigger: always_on
description: This project is a Rust monorepo for deterministic inference and training, focused on MLOps. It replaces the autoregressive text generation of traditional LLMs with a single-forward-pass classification architecture, using dense models from the **Llama, Qwen2, Qwen3, Mistral, Gemma, Gemma2 and Gemma3** families (for example Manacá-1B, Qwen2.5-1.5B-Instruct, the default model), detected automatically from the `model_type` field in `config.json` through the Hugging Face `candle` framework. MoE/MLA f
---

# Guidelines for AI Agents (AGENTS.md)

## 1. Project Context
This project is a Rust monorepo for deterministic inference and training, focused on MLOps. It replaces the autoregressive text generation of traditional LLMs with a single-forward-pass classification architecture, using dense models from the **Llama, Qwen2, Qwen3, Mistral, Gemma, Gemma2 and Gemma3** families (for example Manacá-1B, Qwen2.5-1.5B-Instruct, the default model), detected automatically from the `model_type` field in `config.json` through the Hugging Face `candle` framework. MoE/MLA families (`mixtral`, `qwen3_moe`, `deepseek_v2`, `deepseek_v3`) are rejected with an actionable message.

The goal is to provide an ultra-low-latency API for semantic routing, strictly compatible with the **Jev** (TypeSafe AI) API specification, plus a LoRA/QLoRA/full/from-scratch trainer and a quantization (FP8/FP4) pipeline whose artifacts the server consumes directly.

## 2. Architecture and Technology Stack
*   **Language:** Rust (Edition 2021).
*   **Workspace:** `typed-lm` with three members:
    *   `typed-lm-common` (lib) — Jev contract, labels, prompt rendering, checkpoint and architecture detection, dense-architecture traits, device/dtype (`PrecisionPolicy`), quantization (FP8/FP4), tokenizer.
    *   `typed-lm-serve` (bin) — Actix server; a binary **with no subcommand** (top-level flags).
    *   `typed-lm-trainer` (bin+lib) — subcommands `train` and `quantize`.
*   **Web Server:** `actix-web` with asynchronous concurrency managed by `tokio`.
*   **Inference Engine:** `candle-core`, `candle-nn`, `candle-transformers`. The parametrized dense forward covers all seven dense families (Llama, Qwen2, Qwen3, Mistral, Gemma, Gemma2, Gemma3); the GGUF/GGML-quantized path implements **only Qwen2** and rejects the other architectures at load time.
*   **Training methods (`train`):** `--method lora|qlora|full|from-scratch`. `lora`/`qlora` train adapters over a frozen checkpoint; `full` tunes every parameter from a checkpoint; `from-scratch` initializes every parameter randomly (deterministic through `--seed`). An optional TOML configuration (`--configuration-file`) supplies any parameter with **CLI > TOML > default** precedence; `from-scratch` requires an explicit geometry (`[model]` or flags) and a `tokenizer.json` (`[tokenizer] file` or `--tokenizer-file`).
*   **Module Layout:**
    *   `typed-lm-common/src/architecture_traits.rs` — per-family dense capabilities (attention bias, explicit `head_dim`, sliding window, logit soft-capping, RMSNorm offset, embedding scale, local RoPE).
    *   `typed-lm-serve/src/api/` — response DTOs, errors, routes and Actix handlers.
    *   `typed-lm-serve/src/domain/` — the `Evaluator`/`MockEvaluator` trait.
    *   `typed-lm-serve/src/infrastructure/` — Candle: checkpoint loading, tokenizer, vendored parallel forward and the real evaluator.
    *   `typed-lm-serve/src/config/`, `typed-lm-serve/src/bootstrap/` — CLI and startup/server.
    *   `typed-lm-trainer/src/{dataset,model,training,quantization}/` — training and PTQ pipeline; `model/` includes `initialization`, `trainable_dense`, `trainable_full`, `trainable_linear` and `trainable_rms_norm`, plus the LoRA adapters.
    *   `typed-lm-trainer/src/configuration_file.rs`, `typed-lm-trainer/src/configuration_resolution.rs` — TOML schema and **CLI > TOML > default** resolution.
*   **State Management:** The model, the tokenizer and the `base_cache` (KV-cache of the system prompt) must be loaded once at application startup and shared with the `actix-web` workers through `actix_web::web::Data`. The per-request mutable cache must always be a clone of the base cache.
*   **Supported formats:** safetensors (single or sharded), GGUF (dense and GGML-quantized, e.g. `Q4_K_M`), PyTorch `.pth`/`.bin` and NumPy `.npz`. **FP8 (`F8_E4M3`/`F8_E5M2`) and FP4 (MXFP4) are supported via dequantization at load** to dense F32; `GPTQ`/`AWQ` are rejected with a clear message.
*   **CPU:** the fused `candle-nn` flash attention is used automatically on CPU (keeps GQA grouped); `--features mkl` enables Intel MKL BLAS. GGUF `Q4_K_M` is the recommended CPU mode.
*   **GPU:** `--features cuda` (automatic F16). The devcontainer installs the CUDA toolkit through the `nvidia-cuda` feature and reserves the GPU in `docker-compose.yml`; the host only needs the driver and the NVIDIA container toolkit.
*   **Training precision:** `PrecisionPolicy { master: F32, compute: F32(CPU)/BF16(GPU), reduction: F32 }` — master weight and optimizer in F32; BF16 only as compute on GPU. Functional CPU/CUDA parity, not of speed.
*   **Session cache:** the `system + state` prefix is retained in an LRU cache (by canonical hash of the state, bounded by number of entries and total tokens). The stored value is always a clone; the retained cache is never mutated.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [neurono-ml/typed-lm](https://github.com/neurono-ml/typed-lm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-26 -->
