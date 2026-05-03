---
trigger: always_on
description: MLX-Node is a high-performance machine learning framework for Node.js that brings Apple's MLX library to JavaScript/TypeScript. It supports inference (Qwen3, Qwen3.5, PaddleOCR-VL), training (GRPO, SFT), vision-language models, and document processing pipelines. Uses Metal GPU acceleration through a Rust/NAPI/C++ bridge.
---

# MLX-Node: High-Performance ML Framework for Node.js

## Project Overview

MLX-Node is a high-performance machine learning framework for Node.js that brings Apple's MLX library to JavaScript/TypeScript. It supports inference (Qwen3, Qwen3.5, PaddleOCR-VL), training (GRPO, SFT), vision-language models, and document processing pipelines. Uses Metal GPU acceleration through a Rust/NAPI/C++ bridge.

### Core Technology Stack

- **MLX**: Apple's ML framework with Metal GPU acceleration
- **Rust**: ~100,000 lines across 5 crates (mlx-core, mlx-sys, mlx-paged-attn, mlx-tui, mlx-db)
- **C++**: ~7,700 lines (11 .cpp files + 2 headers) for compiled forward passes and FFI bridge
- **Metal**: ~2,500 lines of custom shaders (paged attention, gated delta recurrence)
- **NAPI-RS**: 328 NAPI exports, 221 FFI bindings
- **TypeScript**: 7,671 source lines across 5 packages + 9,125 test lines
- **Tests**: 1,315 total (900 Rust + 415 TypeScript)

---

## Architecture

```
┌──────────────────────────────────────────────────┐
│  TypeScript Layer (7,671 lines, 5 packages)      │
│  @mlx-node/lm   - Inference, streaming, configs  │
│  @mlx-node/trl  - GRPO/SFT training, datasets    │
│  @mlx-node/vlm  - VLM, document pipelines        │
│  @mlx-node/cli  - Model download, conversion      │
│  @mlx-node/core - Native addon (NAPI bindings)    │
├──────────────────────────────────────────────────┤
│  Rust Compute Layer (~100,000 lines, 5 crates)   │
│  Models: Qwen3, Qwen3.5 Dense/MoE, PaddleOCR-VL │
│  Document: DocLayout, TextDet, TextRec, Ori, Unwarp│
│  Training: GRPO engine, SFT engine, autograd      │
│  Infra: transformers, sampling, tokenizer, KVCache │
│  Paged attention with Metal kernels               │
├──────────────────────────────────────────────────┤
│  C++ Bridge (7,700 lines) → Compiled forward paths│
│  221 FFI functions, compiled decode (mlx::compile) │
├──────────────────────────────────────────────────┤
│  MLX → Metal/Accelerate GPU Backend               │
└──────────────────────────────────────────────────┘
```

### Package Dependency Chain

```
@mlx-node/core (Rust/NAPI native addon - 328 exports)
    ├── @mlx-node/lm   (inference: models, streaming, profiling, tools)
    │     ├── @mlx-node/trl (training: GRPO, SFT, datasets, rewards)
    │     └── @mlx-node/vlm (vision: VLM, OCR, document pipeline)
    └── @mlx-node/cli  (CLI: download, convert)
```

### Rust Crate Inventory

| Crate              | Lines                  | Purpose                                         |
| ------------------ | ---------------------- | ----------------------------------------------- |
| **mlx-core**       | 82,339                 | All NAPI exports: models, training, ops, vision |
| **mlx-paged-attn** | 8,473 + 2,043 Metal    | PagedAttention with Metal kernels               |
| **mlx-tui**        | 5,567                  | Ratatui training TUI (`mlx-train` binary)       |
| **mlx-db**         | 2,345                  | SQLite training output persistence              |
| **mlx-sys**        | 1,148 Rust + 7,717 C++ | Low-level MLX FFI bridge                        |

### mlx-core Key Modules

| Module               | Lines  | Purpose                                                                  |
| -------------------- | ------ | ------------------------------------------------------------------------ |
| `models/`            | 36,867 | 9 model implementations (see Models section)                             |
| `transformer/`       | 9,228  | Attention, KVCache, BatchKVCache, RotatingKVCache, QuantizedKVCache, MLP |
| `utils/`             | 6,981  | GGUF, foreign weights, SafeTensors, functional, pickle, imatrix          |
| `grpo/`              | 6,505  | GRPO/DAPO/Dr.GRPO/BNPO loss, advantages, entropy, engine, rewards        |
| `array/`             | 3,972  | 90+ core ops, padding, masking, thread-safe handles                      |
| `nn/`                | 4,010  | Activations, Linear, Conv1d, RMSNorm, LayerNorm, Embedding, Losses       |
| `vision/`            | 2,178  | Conv2d, interpolate, vision encoder, embeddings, projector, 2D RoPE      |
| `optimizers/`        | 2,203  | Adam, AdamW, SGD, RMSprop                                                |
| `convert.rs`         | 1,765  | Model conversion (dtype, quantization, FP8, recipes)                     |
| `sft/`               | 1,192  | SFT training engine with autograd                                        |
| `output_store/`      | 1,155  | Training output persistence (SQLite)                                     |
| `tools/`             | 1,004  | Tool call/thinking parsing (`<tool_call>`, `<think>` tags)               |
| `tokenizer.rs`       | 1,054  | HuggingFace tokenizers + Jinja2 chat templates                           |
| `sampling.rs`        | 882    | Temperature, top-k/p, min-p, repetition penalty                          |
| `decode_profiler.rs` | 706    | Per-generation profiling with TTFT, memory snapshots                     |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mlx-node/mlx-node](https://github.com/mlx-node/mlx-node) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
