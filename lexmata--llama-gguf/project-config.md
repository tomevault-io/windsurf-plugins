---
trigger: always_on
description: Guidelines for AI coding assistants working on llama-gguf.
---

# AGENTS.md

Guidelines for AI coding assistants working on llama-gguf.

## Project Overview

llama-gguf is a Rust implementation of llama.cpp - a high-performance LLM inference engine. The goal is full feature parity with llama.cpp while providing idiomatic Rust APIs suitable for ecosystem contribution.

## Architecture

Monolithic library with feature flags. Key modules:

| Module | Purpose |
|--------|---------|
| `gguf/` | GGUF file format parsing and writing |
| `onnx/` | ONNX model loading (HuggingFace Optimum exports) |
| `tensor/` | Tensor types, quantization (Q2_K through Q8_0), operations |
| `backend/` | Hardware backends (CPU, CUDA, Metal, DX12, Vulkan) |
| `model/` | Model architectures (LLaMA, Mistral, Qwen2, Qwen3/Qwen3Next, Mixtral, TinyLlama, DeepSeek) |
| `sampling/` | Token sampling strategies (greedy, top-k, top-p, temperature, grammar) |
| `tokenizer/` | BPE tokenizer loaded from GGUF metadata |
| `server/` | HTTP server with OpenAI-compatible API |
| `rag/` | RAG with PostgreSQL/pgvector vector store |
| `model/deltanet.rs` | Gated DeltaNet (SSM) recurrent layers for hybrid models |
| `model/moe.rs` | Mixture-of-Experts routing and expert dispatch |
| `distributed/` | Pipeline-parallel distributed inference via gRPC |
| `huggingface.rs` | HuggingFace Hub model downloading |
| `engine.rs` | High-level inference engine and chat templates |
| `config.rs` | Global configuration |

### Backend Architecture

Each backend implements the `Backend` trait from `backend/mod.rs`:

| Backend | Feature Flag | Platform | Status |
|---------|-------------|----------|--------|
| `cpu/` | `cpu` (default) | All | Production - SIMD optimized (AVX2, AVX-512, NEON) |
| `cuda/` | `cuda` | Linux/Windows | Production - Full GPU-resident inference via `gpu_only.rs` (NVIDIA compute 6.0+) |
| `metal/` | `metal` | macOS | Production - Apple Silicon and AMD GPUs |
| `dx12/` | `dx12` | Windows | Production - DirectX 12 compatible GPUs |
| `vulkan/` | `vulkan` | All | Experimental - Vulkan SDK required |
| `hailo/` | `hailo` | Linux | Experimental - Hailo AI accelerators (Hailo-8L, Hailo-8, Hailo-10H) |

### CUDA GPU-Only Inference

The CUDA backend's primary inference engine is `backend/cuda/gpu_only.rs` (`GpuOnlyInference`). It keeps all model weights, KV cache, and intermediate tensors in VRAM:

- **Quantized weights** are dequantized on-GPU via `dequant_weights.rs`
- **Fused kernels** for RMS norm, RoPE, softmax, SiLU, and element-wise ops
- **DeltaNet** recurrent layers execute entirely on GPU with custom kernels
- **MoE** expert routing and dispatch run on GPU with weight streaming for active experts
- **Attention** uses a hybrid CPU roundtrip for correctness with Qwen3Next-specific features (QK norm, partial RoPE, attention gating)

Other GPU backends (Metal, DX12, Vulkan) use the generic `TransformerLayer::forward()` path from `model/layers.rs`, which dispatches to each backend's `Backend` trait implementation and falls back to CPU for unimplemented operations.

### Hailo Backend (Hybrid CPU+NPU)

The Hailo backend (`backend/hailo/`) offloads transformer subgraphs to Hailo AI accelerators while keeping CPU-bound operations local:

| File | Purpose |
|------|---------|
| `hailo/config.rs` | `HailoConfig`, `HailoQuantization`, `HefManifest` |
| `hailo/context.rs` | `HailoContext` — device management, HEF loading, vstream I/O |
| `hailo/gpu_only.rs` | `HailoGpuInference` — hybrid forward pass orchestrator |
| `hailo/compiler.rs` | ONNX export and DFC auto-compilation to HEF |
| `hailo/mod.rs` | Module root and `Backend` trait stub (all ops unsupported) |

**Execution model** — two HEFs per layer:
- **Attention HEF**: `attn_norm → Q/K/V projections` (runs on Hailo NPU)
- **FFN HEF**: `ffn_norm → gate→SiLU→up→mul→down` (runs on Hailo NPU)
- **CPU**: embedding, RoPE, KV cache, attention scoring, O projection, residual, final norm, logits

**CLI usage**:
```bash
cargo run --release --features hailo -- run model.gguf --hailo --hef-dir /path/to/hefs -p "Hello"
cargo run --release --features hailo -- hailo-info
```

**HEF auto-compile** (requires Hailo DFC Python package):
```bash
cargo run --release --features hailo -- run model.gguf --hailo --auto-compile
```

### RAG Architecture

The `rag/` module provides retrieval-augmented generation backed by PostgreSQL with pgvector:

| File | Purpose |
|------|---------|
| `rag/config.rs` | `RagConfig`, `DatabaseConfig`, `EmbeddingsConfig`, `SearchConfig`, `IndexType`, `DistanceMetric`, `SearchType` |
| `rag/store.rs` | `RagStore` - connection pooling, CRUD, vector/keyword/hybrid search, `MetadataFilter` SQL generation |
| `rag/embedding.rs` | `EmbeddingGenerator` - generates embeddings using a loaded `LlamaModel` |
| `rag/knowledge_base.rs` | `KnowledgeBase` - high-level API for ingestion, chunking, and retrieve-and-generate |
| `rag/mod.rs` | Module exports and `RagError` / `RagResult` types |

Key types:
- `MetadataFilter` - Enum-based filter DSL (Eq, In, Range, Contains, AND/OR/NOT) compiled to parameterized SQL
- `RagContextBuilder` - Builds context strings from search results for prompt injection
- `TextChunker` - Splits documents into overlapping chunks

## Supported Models

Models verified to work correctly:

| Model | RoPE Type | Notes |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Lexmata/llama-gguf](https://github.com/Lexmata/llama-gguf) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
