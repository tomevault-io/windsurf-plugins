---
trigger: always_on
description: Fucina is a close-to-metal **CPU tensor / autograd runtime + LLM inference engine** written in
---

# AGENTS.md — Fucina

Fucina is a close-to-metal **CPU tensor / autograd runtime + LLM inference engine** written in
**Zig 0.16**. North Star: **match or beat llama.cpp on CPU**. It runs Qwen3
dense and the Qwen3-MoE (`qwen3moe`) family, Gemma 4, and several other model families from GGUF
weights (see `docs/RUNNING-MODELS.md`; model weights are not part of the repo). It is CPU-first, with an
optional Metal GPU GEMM offload via `-Dgpu=metal` (see the build options + `src/backend/metal.zig`
below). There is no ggml graph runtime and no C/CMake build — pure Zig vector kernels plus optional
CBLAS for GEMM (the Metal `shim.m`/`.metal` kernels are vendored, not a CMake build).

This file is the working guide for contributors and coding agents: toolchain, build/test commands,
repo map, house rules, and the doc index.

## Toolchain

- Pinned to **Zig 0.16.0** (`zig version` → `0.16.0`).
- No `build.zig.zon` / package manifest — modules are wired directly in `build.zig`.

## Build, test, run, bench

```sh
zig build test                 # unit tests — SEVEN roots: src/fucina.zig, src/llm.zig, examples/nam.zig, examples/parakeet.zig, examples/omnivoice.zig, examples/locate_anything.zig, examples/facedetect.zig
zig build test -Dbackend=scalar   # reference scalar backend
zig build test -Dblas=none        # native backend via pure Zig vector kernels (no CBLAS)
zig build arch-check           # production-only src import graph (AST-based, test-aware): enforces 0 SCCs
zig build doc-check            # doc-index link check: every doc named in AGENTS.md's doc index must exist (tools/check_doc_links.zig)
zig build x86dot-check         # cross-ISA int8/Q4_K/Q8_0/TQ2_0 dot parity checker (follows -Dtarget) + compile-only AVX2/VNNI/smmla bit-rot legs (src/x86dot_check.zig)
zig build cuda-check           # compile-only -Dgpu=cuda leg (x86_64-linux-gnu test root, not run): CUDA-provider bit-rot gate for GPU-less machines
zig build run                  # smoke example (examples/smoke.zig)
zig build qwen3 -- <args>      # Qwen3 GGUF inference (examples/qwen3.zig; --spec/--spec-ref = lossless speculative decode, --tokenize = tokenizer-parity oracle)
zig build gemma4 -- <args>     # Gemma 4 GGUF inference / logit-parity harness; --chat/--repl/--spec (examples/gemma4.zig)
zig build qwen35 -- <args>     # Qwen3.5 (qwen35 hybrid Gated-DeltaNet) GGUF — loader/parity harness (examples/qwen35.zig; see docs/RUNNING-MODELS.md)
zig build parakeet -- <args>   # Parakeet ASR (NeMo FastConformer): WAV → text; --stream/--manifest/--mic (needs -Dparakeet-mic), --compare parity harness (examples/parakeet.zig)
zig build omnivoice -- <args>  # OmniVoice MaskGIT TTS: voice cloning/design/auto, codec encode/decode, parity oracles (examples/omnivoice.zig)
zig build facedetect -- <args> # buffalo_l face pipeline (face-detect.cpp port): info/detect/embed/verify/analyze + bench paired CPU harness (examples/facedetect.zig)
zig build diffusion-gemma -- <args>  # DiffusionGemma block text-diffusion: --eval parity harness vs llama.cpp PR 24423, --chat EB decoding (examples/diffusion_gemma.zig)
zig build locate-anything -- <args>  # LocateAnything-3B open-vocabulary detection: detect/info CLI + exit-code parity gates vs reference dumps (examples/locate_anything.zig)
zig build spirals              # two-spirals training demo: SGD/AdamW/Muon/APOLLO + checkpoints (examples/spirals.zig)
zig build nam -- <args>        # Neural Amp Modeler: .nam profile import/run/train/export, GGUF interchange, live amp sim (examples/nam.zig)
zig build finetune -- <args>   # LoRA fine-tune a Qwen3 GGUF on CPU (examples/finetune.zig)
zig build es-finetune -- <args>  # gradient-free ES fine-tune of a Qwen3 GGUF (examples/es_finetune.zig; --mode lora|full, --reward rule|nll|acc)
zig build es-spirals           # two-spirals MLP trained FROM SCRATCH by ES (examples/es_spirals.zig; self-verifying, member-parallel replicas)
zig build es-ternary-spirals   # two-spirals MLP with PACKED TERNARY (TQ2_0) hidden/output layers trained by ternary-native ES — training state IS the int8 inference model (examples/es_ternary_spirals.zig; see docs/TERNARY.md)
zig build export-gguf -- <args>  # export a GGUF: re-emit/transcode (incl. --dtype tq2_0 ternary), or merge LoRA adapters into dense weights (tools/export_gguf.zig)
zig build bench                # MLP-shaped inference/backward benchmarks
zig build bench-gate           # paired Fucina-vs-llama benchmark gate (tools/bench_gate.py; protocol in docs/BENCHMARK.md)
zig build bench-optim          # optimizer step kernels at LLM shapes (bench/optim.zig)
zig build bench-ce             # softmax / cross-entropy / layerNorm row kernels at LLM shapes (bench/ce.zig)
zig build bench-scatter        # scatter-add (embedding-gradient) kernel at vocab x dim shapes (bench/scatter.zig)
zig build bench-backend        # scalar vs native backends on representative ops
zig build bench-f16gemm        # f16 TransB GEMM parallel-efficiency microbench
zig build bench-gemm           # large-shape f32 GEMM: row kernels vs blocked packed kernel vs BLAS dispatch (bench/gemm.zig)
zig build bench-q5kmoe         # Q5_K MoE-expert matmul: per-row vs 4-row lane-packed col-outer (bench/q5kmoe.zig)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [matteo-grella/fucina](https://github.com/matteo-grella/fucina) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
