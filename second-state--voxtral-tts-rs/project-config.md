---
trigger: always_on
description: Development notes, architecture decisions, and lessons learned during the port of Voxtral-4B-TTS from Python to Rust.
---

# Voxtral TTS Rust -- Implementation Notes

Development notes, architecture decisions, and lessons learned during the port of Voxtral-4B-TTS from Python to Rust.

## Build Commands

```bash
# macOS (MLX — Apple Silicon GPU)
git submodule update --init --recursive
cargo build --release --no-default-features --features mlx

# macOS (libtorch — CPU, for testing/development)
# Download libtorch 2.7.x (must match tch crate version — tch 0.20 requires 2.7)
curl -Lo libtorch.zip https://download.pytorch.org/libtorch/cpu/libtorch-macos-arm64-2.7.1.zip
unzip libtorch.zip
export LIBTORCH=$(pwd)/libtorch
export LIBTORCH_BYPASS_VERSION_CHECK=1
cargo build --release

# Linux (libtorch — CPU or CUDA GPU)
export LIBTORCH=$(pwd)/libtorch
cargo build --release

# Run (libtorch needs library path)
DYLD_LIBRARY_PATH=$LIBTORCH/lib ./target/release/voxtral-tts ...   # macOS
LD_LIBRARY_PATH=$LIBTORCH/lib ./target/release/voxtral-tts ...     # Linux

# Run tests
cargo test

# Run with debug logging
RUST_LOG=debug ./target/release/voxtral-tts models/voxtral-4b-tts --text "Hello." --voice neutral_female --output output.wav
```

## Weight Key Naming

The safetensors checkpoint uses Mistral-style weight naming, **not** HuggingFace convention. This is the most common source of load errors.

### Backbone (237 keys)

| Weight | Key pattern |
|--------|-------------|
| Token embeddings | `mm_audio_embeddings.tok_embeddings.weight` [131072, 3072] |
| Audio codebook embeddings | `mm_audio_embeddings.audio_codebook_embeddings.embeddings.weight` [9088, 3072] |
| Layer attention Q/K/V/O | `layers.{i}.attention.wq.weight`, `.wk.weight`, `.wv.weight`, `.wo.weight` |
| Layer FFN gate/down/up | `layers.{i}.feed_forward.w1.weight`, `.w2.weight`, `.w3.weight` |
| Layer norms | `layers.{i}.attention_norm.weight`, `layers.{i}.ffn_norm.weight` |
| Final norm | `norm.weight` |
| LM head | Absent -- tied to `tok_embeddings.weight` |

### Flow-Matching Transformer (33 keys)

Prefix: `acoustic_transformer.` (NOT `multimodal.acoustic_transformer.`)

| Weight | Key |
|--------|-----|
| Input projection | `acoustic_transformer.input_projection.weight` [3072, 36] |
| LLM projection | `acoustic_transformer.llm_projection.weight` [3072, 3072] |
| Time projection | `acoustic_transformer.time_projection.weight` [3072, 3072] |
| Semantic output | `acoustic_transformer.semantic_codebook_output.weight` [8320, 3072] |
| Acoustic output | `acoustic_transformer.acoustic_codebook_output.weight` [36, 3072] |
| Layers | `acoustic_transformer.layers.{i}.attention.{wq,wk,wv,wo}.weight` |

### Codec Decoder (116 keys)

Prefix: `audio_tokenizer.` (NOT `multimodal.audio_tokenizer.`)

Convolutions use weight normalization with `parametrizations.weight.original0` (g, magnitude [out, 1, 1]) and `parametrizations.weight.original1` (v, direction [out, in, kernel]).

| Weight | Key pattern |
|--------|-------------|
| Decoder conv blocks | `audio_tokenizer.decoder_blocks.{i}.conv.parametrizations.weight.original{0,1}` |
| Decoder transformer layers | `audio_tokenizer.decoder_blocks.{i}.layers.{j}.attention.{wq,wk,wv,wo}.weight` |
| Layer scale | `audio_tokenizer.decoder_blocks.{i}.layers.{j}.attention_scale`, `.ffn_scale` |
| QK norm | `audio_tokenizer.decoder_blocks.{i}.layers.{j}.attention.q_norm.weight`, `.k_norm.weight` |
| Semantic codebook (EMA) | `audio_tokenizer.quantizer.semantic_codebook.embedding_sum` [8192, 256] |
| Cluster usage (EMA) | `audio_tokenizer.quantizer.semantic_codebook.cluster_usage` [8192] |
| Output projection | `audio_tokenizer.output_proj.conv.parametrizations.weight.original{0,1}` |

Decoder block layout (8 blocks total):
- Even indices (0, 2, 4, 6): Conv blocks
- Odd indices (1, 3, 5, 7): Transformer blocks (2 layers each)

## Config Parsing Pitfall

`params.json` stores decoder config as **comma-separated strings**, not JSON arrays:

```json
{
  "decoder_convs_strides_str": "1,2,2,2",
  "decoder_convs_kernels_str": "3,4,4,4",
  "decoder_transformer_lengths_str": "2,2,2,2"
}
```

The code must parse these with `resolve_str_fields()` after serde deserialization. Without this, the codec decoder loads with 0 blocks and the dequantized latent passes straight through to the output projection, causing a channel mismatch.

## MLX Backend -- Critical Lessons

### 1. Lazy Evaluation — eval() at Outer Loop Boundaries Only

MLX builds a computation graph lazily. The graph must be evaluated periodically to prevent unbounded growth, but **over-evaluating kills performance**. Per the MLX documentation and mlx-lm reference implementation, eval() should be called at outer loop boundaries, not per-layer.

**Where eval() is required (current optimized placement):**

- After the full 26-layer backbone forward pass in `forward_one_embedding()` — 1 eval per frame
- After the full 26-layer backbone forward pass in `forward_prefill_embeddings()` — 1 eval total
- After each Euler step in `decode_acoustic()` (flow matching ODE) — 7 evals per frame
- After the full codec decoder (all 4 blocks) in `run_decoder()` — 1 eval per decode

**Where eval() is NOT needed:**
- Per transformer layer (the graph for 26 layers is fine — "thousands of ops" per eval is OK)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [second-state/voxtral_tts_rs](https://github.com/second-state/voxtral_tts_rs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
