---
trigger: always_on
description: Manages sequence lifecycle through states: `WAITING → PREFILLING → DENOISING → SAVING → FINISHED`. Key responsibilities:
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What is JetEngine

JetEngine is a lightweight inference engine for block diffusion language models (SDAR, LLaDa, dLLM-Var). It supports dense and MoE architectures, tensor parallelism, CUDA graph capture, paged KV caching, and flashinfer acceleration. Built from nanovllm.

## Build & Run

```bash
# Install (requires flash-attn, CUDA GPU)
pip install flash-attn --no-build-isolation
pip install .

# Single GPU (must use accelerate or torchrun for distributed init)
CUDA_VISIBLE_DEVICES='0' accelerate launch --multi_gpu example.py
# or
CUDA_VISIBLE_DEVICES='0' torchrun --nproc_per_node=1 example.py

# Multi-GPU (data parallel by default, uses all visible GPUs)
accelerate launch --multi_gpu example.py
```

No test suite exists in this repo. Correctness is verified through end-to-end MATH-500 evaluation (see Evaluation section below).

## Architecture

### Entry Point & Engine

`LLM` (in `jetengine/llm.py`) is a thin subclass of `LLMEngine` (`jetengine/engine/llm_engine.py`). The engine orchestrates the full inference loop:

1. **`add_request()`** — tokenizes prompt, creates a `Sequence`, hands it to the `Scheduler`
2. **`step()`** — calls `scheduler.schedule()` to get prefill/denoise batches, runs them through `ModelRunner`, then postprocesses logits back through the scheduler
3. **`generate()`** — batched generation where all prompts are added upfront, loop `step()` until done
4. **`generate_streaming()`** — streaming generation with backpressure: `max_active` controls how many sequences run concurrently, new prompts are added as old ones finish

### Two Operating Modes

JetEngine operates in two distinct regimes depending on the relationship between total requests and `max_active`:

#### Mode 1: Ideal Decode (total_seqs ≤ max_active)

All sequences fit in memory simultaneously. The flow is:
1. **Prefill phase**: All sequences get prefilled in one batch
2. **Pure denoise phase**: All sequences denoise together with no interruptions
3. **Drain phase**: Sequences finish at different times, batch size shrinks

In this mode, there is no interleaving of prefill and denoise. The chain mechanism (chain=5 when no pending) completes a full block (4 denoising steps + SAVING) in a single scheduler step. This maximizes GPU utilization since the batch stays large.

Profile with: `tests/bench_throughput.py` or `tests/profile_realistic.py`

#### Mode 2: Streaming Decode (total_seqs > max_active)

More sequences than can fit in memory. The engine uses `generate_streaming()`:
1. Initial `max_active` sequences get prefilled
2. Denoise steps run on active sequences
3. As sequences finish and free KV cache blocks, new sequences from the waiting queue get prefilled
4. Prefill and denoise alternate in each `step()` call

In this mode, prefill of new sequences interleaves with denoise of active sequences. The chain depth is limited (chain=2 when prefills are pending) to let the scheduler refill slots. This introduces more scheduling overhead but keeps GPU utilization high.

Profile with: `tests/profile_streaming.py`

The MATH-500 evaluation is an instance of Mode 2: 500 problems × num_generations (e.g. 4) = 2000 total sequences, but max_active is typically 64-128.

### Scheduler (`jetengine/engine/scheduler.py`)

Manages sequence lifecycle through states: `WAITING → PREFILLING → DENOISING → SAVING → FINISHED`. Key responsibilities:
- **Block management** — allocates/deallocates paged KV cache blocks via `BlockManager`
- **Batching** — prepares separate prefill and denoise batches each step
- **Postprocessing** — implements all remasking strategies (sampling + token selection). Has two paths:
  - `postprocess()` — general path, supports per-sequence sampling params and mixed strategies
  - `postprocess_unify()` — optimized batched path when all sequences share the same sampling params

### Chain Mechanism

The chain mechanism runs multiple denoising steps within a single `step()` call, avoiding scheduler overhead between steps. Adaptive chain depth:
- `chain=5` when no pending prefills (full block in one step: 4 denoise + 1 SAVING)
- `chain=4` when batch_size ≤ 8 (streaming tail)
- `chain=2` when prefills are pending (let scheduler refill slots)

Key optimizations in the chain path:
- **Context reuse**: positions/block_tables are cached and reused for pure DENOISING chain steps
- **Zero-sync fast path**: for intermediate chain steps (step < denoising_steps - 1), GPU→CPU synchronization is eliminated by predicting SAVING transitions from CPU-side state
- **Chain batch tokens**: batch tensor from postprocess is passed directly to prepare_denoise, avoiding per-sequence Python loop + torch.cat

### Sequence (`jetengine/engine/sequence.py`)

Represents a single generation request. Tracks block diffusion state: `intermediate_block_tokens` (current block being denoised as a tensor), `block_trajectory`, `block_logprobs`, `block_entropies`. The `commit_block()` method finalizes a denoised block into the sequence's token stream.

### ModelRunner (`jetengine/engine/model_runner.py`)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Labman42/JetEngine](https://github.com/Labman42/JetEngine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
