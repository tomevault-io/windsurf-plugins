---
trigger: always_on
description: imp was built with [Claude Code](https://claude.ai/claude-code) (mostly Opus) as a proof of concept — every line of C++, every CUDA kernel, every optimization came out of AI-human collaboration via the CLI agent. Sonnet handled small refactors and review when Opus was unavailable; Opus did the heavy work (kernel design, dispatch refactors, multi-week investigations like the FP8 KV warmup-calibration bug or the Qwen3.6-NVFP4 decode fast-path).
---

# AGENTS.md

## How This Project Was Built

imp was built with [Claude Code](https://claude.ai/claude-code) (mostly Opus) as a proof of concept — every line of C++, every CUDA kernel, every optimization came out of AI-human collaboration via the CLI agent. Sonnet handled small refactors and review when Opus was unavailable; Opus did the heavy work (kernel design, dispatch refactors, multi-week investigations like the FP8 KV warmup-calibration bug or the Qwen3.6-NVFP4 decode fast-path).

### The Process

1. **Architecture first.** Modular source tree (`core/`, `compute/`, `memory/`, `model/`, `quant/`, `graph/`, `runtime/`, `vision/`, `api/`), C-compatible public API, and a hardcoded forward pass — no runtime graph walker. Early shape choices paid dividends throughout.

2. **Kernel-by-kernel, profile-driven.** Every kernel: scalar reference first, then WMMA tensor-core, then CUTLASS SM120 (FMHA in FP16 / FP8 / MXFP4, NVFP4 grouped GEMM, MoE grouped GEMM). Benchmarked with `nsys profile --stats=true` before moving on. Specialised exclusively for `sm_120f` — there's no compatibility matrix to maintain across architectures.

3. **NVFP4 as the Blackwell path.** The model-optimizer / llm-compressor SafeTensors loaders, the contiguous per-expert NVFP4 buffer (`cache_moe_native_nvfp4`), and the decode fast-path were built specifically to make Blackwell's FP4 tensor cores (3354 TOPS) and graph-capturable expert dispatch work end-to-end. GGUF support came along for the ride — Q8_0 first because simplest, then K-quants — but the headline numbers come from the NVFP4 path.

4. **dp4a GEMV for GGUF decode.** Single-token GGUF decode is memory-bound. The dp4a (INT8 dot product) GEMV kernels use two strategies: K-parallel (1 row/block, all SMs) for small matrices, row-parallel (NR rows/warp, shared-memory cached activations) for large matrices. A runtime heuristic selects the better strategy per layer.

5. **Hard data over intuition.** When template consolidation caused a 2–5% regression, nsys identified the bottleneck kernels. When a fused QKV bias kernel caused a 3.5% regression under CUDA Graphs, nsys proved that 3 small independent kernels pipeline better than 1 merged kernel. Performance work in this repo gets benchmarked, not asserted.

6. **MoE and hybrid architectures.** MoE (Mixtral, DeepSeek, Qwen3-MoE, Qwen3-Coder, Qwen3.6, Gemma-4) and hybrid GDN + Attention + MoE (Qwen3.5/3.6) and Mamba2 + Attention + MoE (Nemotron-H) were added iteratively. Custom fused MoE kernels with shared-memory expert caching outperform llama.cpp by 12–134% on the GGUF path; the NVFP4 contiguous-expert decode path closes the gap further (Qwen3-Coder NVFP4 272 tok/s, Gemma-4 NVFP4 213 tok/s).

7. **Testing throughout.** ~700 Google Tests across 8 binaries cover tensor ops, GGUF + SafeTensors loaders, KV cache, attention kernels (paged decode, FMHA), RoPE, LayerNorm, MoE routing, quantization (FP8 / NVFP4 / MXFP4), tokenizer round-trip including special-token pre-split, chat templates, and end-to-end generation. Tests are written alongside the code, not after.

### Hard-Won Lessons

- **Never `__noinline__` GPU inner-loop functions.** GPU device function calls spill to Local Memory (DRAM). On Q6_K with 54 calls/thread at K=13824, `__noinline__` on `dp4a_block` caused a 39.5% regression. All dp4a specializations must stay `__forceinline__`.
- **CUDA Graphs change fusion calculus.** Without graphs, fusing 3 small kernels into 1 saves launch overhead. With graphs, the 3 independent kernels pipeline better. Always benchmark with graphs enabled.
- **RTX 5090 (sm_120) has only 100 KB shared memory per SM**, not 228 KB like H100. Kernels that assume Hopper shared memory sizes will silently fail or underperform.
- **WSL2 mapped pinned memory** requires explicit `cudaStreamSynchronize`. GPU writes via `cudaHostAllocMapped` are not immediately visible to host — `__atomic_load_n` is insufficient.
- **Q8_0 blocks are 34 bytes** (not 4-aligned). `reinterpret_cast<const int32_t*>` causes misaligned address CUDA errors. Use `memcpy()` instead.

---

## Contributing with AI Agents

This project welcomes contributions from AI coding agents. If you're an AI agent (Claude Code, Cursor, Copilot, Aider, or others) working on this codebase, follow these rules.

### Rules

1. **Read before you write.** Always read the files you intend to modify. Understand the existing patterns before changing anything. The codebase has consistent conventions — follow them.

2. **Build and test before committing.** Every change must:
   ```bash
   make build                # Docker imp:test image (canonical workflow)
   make test-gpu             # Full GTest suite, all binaries
   make verify-fast          # Build + filtered tests + perf gate + smoke prompt (~90s)
   ```
   If you add new functionality, add tests for it in `tests/`. The pre-push

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kekzl/imp](https://github.com/kekzl/imp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
