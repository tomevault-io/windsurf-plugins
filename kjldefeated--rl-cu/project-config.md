---
trigger: always_on
description: - **Phase 1 COMPLETE (correctness)**: LLMEngine all 11/11 tests pass, Sampler kernel done
---

# GRPO-CUDA Project Memory

## Current Status (as of 2026-04-07)
- **Phase 1 COMPLETE (correctness)**: LLMEngine all 11/11 tests pass, Sampler kernel done
- **Throughput (256 seqs)**: 1356 tok/s (up from 821 tok/s baseline) — ~18% of nano-vllm target
- **Next step 1**: Close remaining throughput gap (Flash Decoding, fused norm+proj)
- **Next step 2**: Phase 2 SFT training infrastructure

## Optimizations Applied (see docs/ENGINE.md for details)
1. **CUDA graph execution** — decode buckets 1,2,4,8..256 captured and replayed (DONE)
2. **Gumbel-max sampler v3** — single-pass argmax(logit/T + Gumbel), 51 µs vs 679 µs (DONE)
3. **Continuous batching** — two-phase decode+prefill per step, avg batch now tracks queue depth (DONE)
4. **Fused QKV projection** — 3 GEMMs → 1 per layer (qkv_proj alias, zero extra memory) (DONE)
5. **Fused gate+up projection** — 2 GEMMs → 1 per layer (gate_up_proj alias) (DONE)

## Remaining Gap vs nano-vllm (7411 tok/s)
Priority areas:
1. **Flash Decoding (split-K)**: current paged decode is 1 thread/(seq,head) — expected 5–10× attn speedup
2. **Fused RMSNorm + linear**: avoid [T,1024] HBM write/read between norm and GEMM (2 fusion points/layer)
3. **cuBLAS GEMM tuning**: thin-matrix performance at small B
6. **Continuous batching**: Verify scheduler fills GPU efficiently; check batch padding overhead
7. **Flash attention decode**: Replace single-thread decode with a proper FlashDecoding kernel (split-K)

## Project: grpo-cuda — Pure C++/CUDA LLM RL Engine
Target model: Qwen3-0.6B (FP16, inference phase 1)

## GPU / Build
- GPU: sm_120 (Blackwell, RTX PRO 6000 / RTX 50xx)
- CUDA: 12.8 at /usr/local/cuda-12.8
- Build: `make test_attention` (or `make tests` for all)
- Arch flag: `--gpu-architecture=sm_120`

## Qwen3-0.6B key dims
- hidden=1024, layers=28, H_q=16, H_kv=8, head_dim=128
- GQA ratio: 2:1 (h_kv = h_q * H_kv / H_q)
- QK-Norm BEFORE RoPE (not after — HuggingFace order)

## Kernel status
| Kernel      | File                              | Status     |
|-------------|-----------------------------------|------------|
| RMSNorm     | src/kernels/rmsnorm.cu            | DONE, passing |
| SwiGLU      | src/kernels/swiglu.cu             | DONE, passing |
| Softmax     | src/kernels/softmax.cu            | DONE, passing |
| FA2 Prefill | src/kernels/attention.cu          | DONE, passing |
| Paged Attn  | src/kernels/attention.cu          | DONE, passing |
| KV Cache    | src/model/kv_cache.cu             | DONE, passing |
| RoPE        | src/kernels/rope.cu               | DONE, passing |
| Embedding   | src/kernels/embedding.cu          | DONE, passing |
| Sampler     | src/kernels/sampler.cu            | DONE, passing |

## Sampler design (src/kernels/sampler.cu)
- API: launch_sampler(const half* logits, float* temp_probs, num_tokens, vocab_size,
        top_k, top_p, temperature, seed, int* output_ids, stream)
  - temp_probs: FP32 workspace [num_tokens, vocab_size], pre-allocated by caller
  - MAX_SAMPLER_TOP_K = 1024 (clamped in wrapper)
- Grid: (num_tokens,); Block: 256; Dynamic smem: top_k*8 + 256*8 bytes
- temperature==0 fast path: single argmax pass, return (no softmax)
- Steps: temp-scale + softmax → k-rounds argmax (zeroing selected prob) →
  top-p nucleus (renormalize within top-k first, then cumsum walk) →
  splitmix64 hash → weighted sample
- Bench (V=151936): greedy=14 µs, top-k=1: 113 µs, top-k=50 B=1: 679 µs, B=16: 710 µs
- Test tie policy: use threshold-based top-k set (includes all tied boundary tokens)
  and value-equality for greedy (FP16 ties are common with random logits at large V)

## Attention kernel design
- FA2 prefill: template<HEAD_DIM=128, Br=16, Bc=64>
  - Grid: (ceil(S/Br), H_q, B); Block: (Br)
  - Shared memory: K_smem[Bc][HEAD_DIM] + V_smem[Bc][HEAD_DIM] = 32 KB
  - Registers: q_reg[128], o_acc[128] in FP32; causal break in inner loop
  - Online softmax: alpha = exp(row_max - new_max); p = exp(dot - new_max)
  - CRITICAL: NO early return (if q_row>=S return) inside the kv_tile loop!
    All Br threads must cooperate in K/V loading and reach __syncthreads().
    Use `const bool valid_q = (q_row < S)` and guard only the score+write.
    Bug: when last q_tile has mixed valid/invalid threads, early-returning
    threads skip loading their K_smem rows → garbage dot products → NaN for S>Br.
- Paged decode: template<HEAD_DIM=128, BLOCK_SIZE=16>
  - Grid: (num_seqs, H_q); Block: (1) — single thread per (seq, head)
  - Walks block_tables[seq][logical_block] → physical_block index
  - k_cache layout: [num_blocks, H_kv, BLOCK_SIZE, D]

## KV Cache design
- KV_BLOCK_SIZE = 16 (tokens per physical block)
- Pool layout: [num_layers, total_blocks, num_kv_heads, KV_BLOCK_SIZE, head_dim]
- slot_mapping[token] = physical_block * KV_BLOCK_SIZE + in_block_offset (int64)
- reshape_and_cache_half_kernel: Grid(num_tokens,1), Block(128,1), stride over (head,dim)
- paged_kv_cache_append_slot() builds slot on CPU; returns slot for slot_mapping
- paged_kv_cache_fork() copies block-table pointers only (no KV copy) for GRPO
- PagedKVCache struct in include/model/kv_cache.cuh

## Test tolerances (from DESIGN.md)
- RMSNorm, RoPE, SwiGLU: max_err < 1e-3
- FA2 attention: max_err < 5e-3 vs naive attention

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [KJLdefeated/RL.cu](https://github.com/KJLdefeated/RL.cu) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
