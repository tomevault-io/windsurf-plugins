---
trigger: always_on
description: direct-commits-allowed: true
---

# CLAUDE.md

direct-commits-allowed: true

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Voxtral Mini 4B Realtime — streaming ASR and TTS in Rust using Burn. Runs natively (Vulkan/Metal) and in the browser (WASM + WebGPU). The Q4 GGUF ASR path enables a 4B-parameter model to run entirely client-side in a browser tab. The TTS pipeline uses Voxtral 4B TTS with 20 preset voices across 9 languages.

## Build & Development Commands

```bash
# Native build
cargo build --features "wgpu,cli,hub"
cargo build --release --features "wgpu,cli,hub"

# WASM build (produces pkg/ directory)
wasm-pack build --target web --no-default-features --features wasm

# Lint both targets
cargo clippy --features "wgpu,cli,hub" -- -D warnings
cargo clippy --no-default-features --features wasm --target wasm32-unknown-unknown -- -D warnings

# Format
cargo fmt
cargo fmt -- --check

# Tests (230 tests: 227 unit + 3 integration)
cargo test --features "wgpu,cli,hub"
cargo test test_q4_roundtrip_small           # single test
cargo test gguf::tests                        # module tests

# Benchmarks (Criterion)
cargo bench --features "wgpu,cli,hub" q4_ops       # Q4 matmul kernel microbenchmarks
cargo bench --features "wgpu,cli,hub" q4_pipeline   # full inference pipeline (requires GGUF model)
cargo bench --features "wgpu,cli,hub" audio          # CPU audio processing

# E2E benchmark binary (stage-level timing, JSON output)
cargo run --features "wgpu,cli,hub" --bin e2e-bench -- \
  --audio test_data/mary_had_lamb.wav \
  --gguf models/voxtral-q4.gguf \
  --tokenizer models/voxtral/tekken.json

# E2E browser test (requires model shards + Playwright)
bunx playwright test tests/e2e_browser.spec.ts

# CLI ASR inference
cargo run --features "wgpu,cli,hub" --bin voxtral -- \
  transcribe --audio test_data/mary_had_lamb.wav --gguf models/voxtral-q4.gguf

# CLI TTS inference (BF16)
cargo run --features "wgpu,cli,hub" --bin voxtral -- \
  speak --text "Hello world" --voice casual_female

# CLI TTS inference (Q4, real-time)
cargo run --features "wgpu,cli,hub" --bin voxtral -- \
  speak --text "Hello world" --gguf models/voxtral-tts-q4.gguf --euler-steps 3

# Dev HTTPS server for browser testing
bun serve.mjs
```

## Feature Flags

```toml
default = ["wgpu", "native-tokenizer"]
wgpu              # Burn GPU backend (WebGPU/Vulkan/Metal) — required for gguf module
native-tokenizer  # Tekken BPE encoding via tiktoken-rs (WASM-compatible)
wasm              # WASM bindings: enables wgpu + wasm-bindgen + web-sys + dep:wgpu
cli               # clap + indicatif for voxtral-transcribe and voxtral-speak binaries
hub               # hf-hub for model downloads
```

The `wasm` feature enables both `wgpu` (for burn's WebGPU backend) and `dep:wgpu` (direct wgpu crate access for custom device init). The `gguf` module is gated behind `wgpu`. The `web` module is gated behind `wasm`.

## Model Weights

```bash
# BF16 SafeTensors (~9 GB)
uv run --with huggingface_hub hf download mistralai/Voxtral-Mini-4B-Realtime-2602 --local-dir models/voxtral

# Q4 GGUF (~2.5 GB)
uv run --with huggingface_hub hf download TrevorJS/voxtral-mini-realtime-gguf voxtral-q4.gguf --local-dir models

# TTS model (~8 GB, includes voice presets)
uv run --with huggingface_hub hf download mistralai/Voxtral-4B-TTS-2603 --local-dir models/voxtral-tts

# Sharded for WASM (5 × ≤512 MB each)
# Expected: models/voxtral-q4-shards/shard-{aa..ae}
```

## Architecture

### Data Flow

```
Audio (16kHz) → Peak normalize (0.95) → Chunk if needed (max 1200 mel frames)
  → Mel [B, 128, T] → Encoder [B, T/4, 1280] → Reshape [B, T/16, 5120]
  → Adapter [B, T/16, 3072] → Decoder (autoregressive) → Token IDs → Text
```

### Audio Peak Normalization

Both the CLI (`src/bin/transcribe.rs`) and WASM (`src/web/bindings.rs`) paths call `AudioBuffer::peak_normalize(0.95)` before mel computation. This is critical for Q4 inference: quiet audio (peak < 0.02) produces mel spectrograms indistinguishable from silence after log normalization with `log_mel_max=1.5`, causing the decoder to emit only pad tokens. The f32 model tolerates this due to higher precision throughout the pipeline, but Q4 cannot resolve the subtle mel features. Normalization to 0.95 lifts quiet audio ~4.75 in log-space while barely affecting normal audio (~0.05 shift).

### Two Inference Paths

**BF16 path** (`src/models/`): SafeTensors weights, full precision, native only.

**Q4 GGUF path** (`src/gguf/`): Q4_0 quantized weights, fused GPU dequant+matmul via custom WGSL shader, works in browser.

| Component | BF16 | Q4 GGUF |
|-----------|-----|---------|
| Weight source | SafeTensors | GGUF v2/v3 |
| Linear layers | Burn tensor matmul | Custom `q4_matmul` (shader.wgsl) |
| Token embeddings | f32 tensor (1.5 GiB) | Q4 on GPU + CPU bytes for lookups |
| Memory (model) | ~16 GB | ~2.5 GB |

### GGUF Pipeline

```
reader.rs (GGUF parser + ShardedCursor)
  → loader.rs (Q4ModelLoader, two-phase for WASM)
    → model.rs (Q4 model components, TokEmbedStore)
      → tensor.rs + linear.rs + op.rs + shader.wgsl (GPU Q4 matmul)
        → bindings.rs (WASM JS API)
```

### Q4 Shader Architecture

Two WGSL kernel variants in `src/gguf/`:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TrevorS/voxtral-mini-realtime-rs](https://github.com/TrevorS/voxtral-mini-realtime-rs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
