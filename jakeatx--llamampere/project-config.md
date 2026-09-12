---
trigger: always_on
description: This repo is a fork of [llama.cpp](https://github.com/ggml-org/llama.cpp) (upstream) that adds the TurboQuant feature set on top of a fully-synced upstream base. The local tree always contains all of upstream master plus fork additions; rebasing onto latest upstream is a recurring task.
---

# Instructions for llama.cpp (TurboQuant fork)

## Project Overview

This repo is a fork of [llama.cpp](https://github.com/ggml-org/llama.cpp) (upstream) that adds the TurboQuant feature set on top of a fully-synced upstream base. The local tree always contains all of upstream master plus fork additions; rebasing onto latest upstream is a recurring task.

### What TurboQuant adds

TurboQuant compresses the KV cache far beyond the standard `q8_0` by applying a fixed 128x128 orthonormal Walsh-Hadamard rotation (`GGML_OP_TURBO_WHT`) to cache vectors before quantization, which Gaussianizes the distribution, and inverse-rotating after dequantization. Head dims that are not multiples of 128 are zero-padded. MLA models (DeepSeek) have no separate V cache, so V rotation/padding is skipped for them, and K/V cache types must be identical.

The five fork-only GGML types (registered in `ggml/include/ggml.h`):

| Type   | Enum                       | Purpose                             | Size         |
|--------|----------------------------|-------------------------------------|--------------|
| turbo2 | `GGML_TYPE_TURBO2_0` (43)  | KV cache only                       | 2 bits/value |
| turbo3 | `GGML_TYPE_TURBO3_0` (44)  | KV cache only                       | 3.25 bits    |
| turbo4 | `GGML_TYPE_TURBO4_0` (47)  | KV cache only                       | 4.25 bits    |
| TQ3_1S | `GGML_TYPE_TQ3_1S` (45)    | model weights, WHT-rotated Lloyd-Max| 3 bits, block 32 |
| TQ4_1S | `GGML_TYPE_TQ4_1S` (46)    | model weights                       | 4 bits, block 32 |

Turbo cache types are runtime-only, never stored in GGUF. TQ3_1S/TQ4_1S are first-class weight types with CPU, CUDA/HIP (warp-cooperative mmvq), Metal, Vulkan, and SYCL kernels, exposed as `llama-quantize` targets.

### Key files

- `ggml/src/ggml-turbo-quant.c` - the codec (keep byte-identical to fork tip)
- `ggml/include/ggml.h` - type enum 43-47, `GGML_OP_TURBO_WHT`
- `src/llama-kv-cache.cpp` - cache wiring, `get_k_idx`, layer-adaptive precision
- `src/llama-graph.cpp` - inverse-WHT post-processing (FA and non-FA paths)
- `ggml/src/ggml-cuda/mmvq-tq.cu` - native TQ dp4a kernels (`GGML_TQ_NATIVE=1`)
- `ggml/src/ggml-vulkan/` - turbo FA, SET_ROWS, dequant shaders
- `ggml/src/ggml-metal/ggml-metal.metal` - TurboFlash kernels
- `docs/KV-cache-quantization.md` - authoritative usage doc (read before touching cache types)

### Usage

```bash
llama-cli -m model.gguf -c 8192 -ngl 99 --cache-type-k q8_0 --cache-type-v turbo3
```

Any combination of `f16`/`q8_0`/`turbo2`/`turbo3`/`turbo4` for K and V is supported. Turbo cache types require flash attention; it is auto-enabled with a warning. Quantized V with FA explicitly disabled is an error (upstream behavior). The same flags work in `llama-server`, `llama-bench`, `llama-perplexity`.

### Environment knobs

| Variable                    | Default | Effect |
|-----------------------------|---------|--------|
| `TURBO_LAYER_ADAPTIVE`      | `0`     | Layer-adaptive KV precision; `7` = Boundary V (edge layers q8_0, middle turbo) |
| `TURBO_AUTO_ASYMMETRIC`     | `1`     | Auto-select asymmetric K/V types for large-GQA models |
| `TURBO_SPARSE_V`            | `1`     | Sparse-V dequant skip in flash attention |
| `GGML_TQ_NATIVE`            | unset    | `1` opts out of load-time TQ->q8_0 conversion, uses fused native TQ kernels (saves ~1.7x VRAM on decode-heavy workloads) |
| `LLAMA_ATTN_ROT_K/V_OVERRIDE` | off   | Optional upstream attention rotation (TurboQuant manages its own rotation) |

### Test gates (all must pass before touching quant/backend code)

- `test-turbo-quant` - turbo3 basis MSE=0/Cosine=1.0, turbo4 Cosine=0.9956
- `test-quantize-fns` - includes TQ3_1S/TQ4_1S and rotated-domain buffer sizing
- `test-backend-ops` - full sweep on CPU + CUDA0 (23k+ cases on the RTX 5090 dev box)
- `llama-bench` with `-ctk/-ctv turboN`; type parser accepts `tq3_1s`/`tq4_1s`

### What the test gates do and do not cover

What each suite does:

- `test-turbo-quant` - CPU codec round-trip quality: quantize -> dequantize -> CPU inverse WHT, MSE/cosine on fixed vectors, plus a chunked-dequant invariance check for all five turbo types at row lengths straddling the vec_dot chunk size. No GGML graphs, no backend kernels.
- `test-quantize-fns` - CPU quantize/dequantize functions against error budgets, including TQ3_1S/TQ4_1S. Skips TURBO2_0/3_0/4_0 by design: their dequant output stays in the WHT-rotated domain.
- `test-backend-ops` - per-op GGML graphs, run on each backend and compared numerically against the CPU reference. This is the only gate that exercises backend kernels.
- `llama-bench` - tokens/s on real models. Timing only; it never checks output correctness.

Coverage limits (each caused a real miss):


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JakeATX/llamAmpere](https://github.com/JakeATX/llamAmpere) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-12 -->
