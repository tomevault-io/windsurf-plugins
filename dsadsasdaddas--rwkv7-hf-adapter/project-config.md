---
trigger: always_on
description: This repository is now scoped to the **RWKV-7 Hugging Face / Transformers adapter only**.
---

# AGENTS.md

## Project Mission

This repository is now scoped to the **RWKV-7 Hugging Face / Transformers adapter only**.

Historical upstream context mentioned three independent tracks: Hugging Face,
vLLM, and SGLang. For this repository and active work, do **not** build or gate
native vLLM/SGLang integrations. Any vLLM/SGLang work is a separate future
project and must not block HF deliverables.

The active reward target is the HF/Transformers track: make RWKV-7 usable from
standard HF APIs with near-production correctness, performance, memory behavior,
training compatibility, quantized inference, and reproducible benchmarks.

## Current Agent Contract: Native Fused HF Backend

This is the active contract for the next workers on this branch. Treat
`FUSED_BACKEND.md` and `docs/native_fused_roadmap.md` as the performance
roadmap.

- Scope is **HF adapter only**. Do not implement or gate native vLLM/SGLang
  integrations in this repository.
- Keep HF compatibility as the invariant: `AutoModelForCausalLM`, `generate`,
  PEFT, Trainer, TRL, `RWKV7StateCache`, dynamic batching, chunked prefill,
  save/load, and quantized loading must keep working.
- Move the speed core to native fused backends: fused fp16 first, then fused
  quant. The wrapper is the compatibility shell, not the place for the next
  performance breakthrough.
- Use official math alignment from `rwkv_v7_numpy.py` and
  `run_rwkv7_qwen35.py`; preserve exact RWKV-7 recurrence, clamp, state, and
  output semantics before optimizing layout.
- Follow train_temp-style fused boundaries: `tmix_mix6`, `kk_pre/state_prep`,
  `lnx_rkvres_xg`, `cmix`, and `clampw`.
- Use Albatross-style GPU-specific layout/autotune. Exact-card rows decide
  defaults; V100, 4090, A100/H100, and Blackwell must not blindly share tile
  choices.
- Treat DPLR/chunked prefill as the bsz=1 prompt-prefill breakthrough line.
  Do not spend the next phase on wrapper/cache micro-optimizations.
- Forbidden directions: wrapper micro-optimization as the main plan, native
  vLLM/SGLang work, defaulting the full-head scan+output fused prefill path,
  and quantized-speed claims before a native fused quant kernel beats fp16
  end-to-end.
- Required next validation loop: RTX 4090 fp16, bsz=1/4, prompt512 prefill,
  decode, correctness, peak memory/VRAM, and `bench/analyze_results.py`
  reporting.

## Current Branch Goal: DPLR/WY Compiled Prefill Prototype

Active branch work is now the opt-in DPLR/WY compiled prefill backend, not
wrapper micro-optimization. Keep the default HF behavior unchanged unless a
benchmark explicitly opts in.

Goal:

- Move `dplr_chunk_scan(algorithm="wy"/"lowrank")` from pure torch
  correctness prototype toward a real Triton/CUDA performance prototype.
- Maintain native VxK state layout `[B,H,N,N]`, fp16/bf16/fp32 token inputs,
  and fp32 state accumulation.
- Synthetic first: support the critical target
  `B=1,H=16,N=64,T=512,chunk_size=64,fp16` on RTX 4090.
- Correctness gates: match `torch_recurrent_scan`; for fp16 target require
  `out_min_cosine >= 0.9999` and keep greedy/cache smoke passing when routed
  through HF repo-code loading.

Current implementation state:

- `rwkv7_hf/dplr_prefill_triton.py` exposes:
  - `dplr_chunk_scan_triton(...)` / `dplr_chunk_scan_triton_available()`
  - dense chunk summary: `dplr_dense_chunk_summary_*`
  - dense prefix combine: `dplr_dense_prefix_combine_*`
  - dense chunk apply/output: `dplr_dense_chunk_apply_*`
  - dense three-stage scaffold: `dplr_dense_three_stage_triton(...)`
- `algorithm="triton_wy"` is the P0 compiled bridge using the existing fused
  recurrent scan. It is fast and correctness-passing, but it is not yet compact
  WY.
- `algorithm="triton_dense3"` is the explicit dense three-stage scaffold
  (summary -> prefix -> apply/output). It proves the mathematical kernel
  boundaries, but it materializes dense `[N,N]` summaries and is expected to be
  slower than the P0 fused scan until replaced with compact WY factors.

Latest RTX 4090 target evidence:

- Synthetic `B=1,T=512,H=16,N=64,chunk=64,fp16`:
  - `sequential`: pass, about `55.63 ms`, `9.2k tok/s`
  - `triton_wy`: pass, about `0.233 ms`, `2.20M tok/s`,
    `out_min_cosine ~= 0.9999999`
  - `triton_dense3`: pass; latest stage-probe full row is about
    `0.264-0.269 ms`, `~1.9M tok/s`, `out_min_cosine = 1.0`
  - dense stage split from `--stage-probe`: summary `~0.144 ms`, prefix
    `~0.092 ms`, apply/output `~0.065 ms`, summary shape
    `[1,8,16,64,64]`. Dense summary/prefix `[N,N]` traffic is the first
    compact-WY target.
- HF repo-code smoke on 4090 / 0.4B / prompt512 / bsz1:
  - Sweep path: `/tmp/native_4090_todo_sweep_20260702_103919.jsonl`.
  - Albatross reference for this shape remains `52,148.52 tok/s`; `0.45x`
    is `23,467 tok/s`.
  - DPLR repo-code rows: `triton_wy` pass at `20,421.7 tok/s` (`0.3916x`),
    `triton_dense3` pass at `18,546.0 tok/s` (`0.3556x`),
    `triton_wy_compact` pass at `17,970.5 tok/s` (`0.3446x`).
  - Fastest short sweep row was the fused recurrent scan path, not DPLR:
    `RWKV7_NATIVE_PREFILL_FUSED_SCAN=1`,
    `RWKV7_NATIVE_PREFILL_SCAN_BLOCK_M=8`,
    `RWKV7_NATIVE_PREFILL_SCAN_NUM_WARPS=1`,
    `RWKV7_NATIVE_PREFILL_FUSED_STATE_PREP=1`; pass at `22,777.0 tok/s`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dsadsasdaddas/rwkv7-hf-adapter](https://github.com/dsadsasdaddas/rwkv7-hf-adapter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-03 -->
