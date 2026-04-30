---
trigger: always_on
description: Pure Rust implementation of Qwen3-ASR (automatic speech recognition) with two backends:
---

# Qwen3-ASR-RS Development Guide

## Project Overview

Pure Rust implementation of Qwen3-ASR (automatic speech recognition) with two backends:
- **libtorch** (via `tch` crate) — cross-platform, optional CUDA
- **MLX** (Apple Silicon) — Metal GPU via `mlx-c` submodule

Loads safetensors weights directly and re-implements the full neural network forward pass.

Git DCO: `Signed-off-by: Michael Yuan <michael@secondstate.io>`

## Build Commands

```bash
# macOS (MLX backend — recommended for Apple Silicon)
git submodule update --init --recursive
cargo build --release --no-default-features --features mlx,build-ffmpeg

# Linux (libtorch backend)
export LIBTORCH=$(pwd)/libtorch
export LIBTORCH_BYPASS_VERSION_CHECK=1
cargo build --release --features build-ffmpeg
```

Features are mutually exclusive: `tch-backend` (default) XOR `mlx`. Enforced at compile time in `lib.rs`.

## Architecture

Pipeline: **Audio → FFmpeg decode → 16kHz mono → Mel spectrogram → Audio encoder → Text decoder → Transcription**

### Model Architecture (Qwen3-ASR-0.6B defaults)

**Audio Encoder** (Whisper-style, `audio_encoder.rs`):
- 3x Conv2d downsampling (stride 2, 8x total reduction)
- Sinusoidal positional embeddings (computed at runtime, not stored in weights)
- 18 transformer layers (d_model=896, 14 attention heads)
- Output projection: LayerNorm → Linear → GELU → Linear (896 → 1024)
- Chunk-based: splits mel into n_window*2 (100) frame segments, processes ALL as a single batch

**Text Decoder** (Qwen3, `text_decoder.rs`):
- 28 transformer layers (hidden_size=1024, head_dim=128)
- GQA: 16 Q heads, 8 KV heads
- QK-normalization via per-head RMSNorm
- MRoPE (Multimodal Rotary Position Embeddings), section sizes [24, 20, 20]
- SwiGLU MLP (gate_proj → silu, up_proj, down_proj)
- Tied word embeddings (embed_tokens = lm_head by default)

### Key File Map

| File | Purpose |
|------|---------|
| `tensor.rs` | Unified Tensor abstraction over tch/MLX (largest file, ~1200 lines) |
| `layers.rs` | All transformer building blocks (norms, linear, attention, MLP, MRoPE) |
| `inference.rs` | End-to-end ASR pipeline orchestration |
| `audio_encoder.rs` | Whisper-style encoder with chunk-based Conv2d processing |
| `text_decoder.rs` | Qwen3 decoder with KV cache |
| `mel.rs` | Whisper-style 128-bin log-mel spectrogram (Slaney scale) |
| `audio.rs` | FFmpeg audio loading + WAV/rubato fallback |
| `weights.rs` | Safetensors loading (single-file + sharded, bf16/f16 → f32) |
| `config.rs` | Model configuration deserialization |
| `tokenizer.rs` | HuggingFace tokenizer wrapper |
| `backend/mlx/` | MLX C FFI bindings, safe array wrappers, ops, signal processing |

## Critical Implementation Details

### Mel Spectrogram (`mel.rs`)
- Uses **Slaney** mel scale (NOT HTK): linear below 1000 Hz, logarithmic above
- STFT requires center padding via `reflection_pad1d` (n_fft//2 on each side)
- Must remove last STFT frame: `magnitudes[..., :-1]`
- Pad audio samples to next multiple of hop_length
- Log-mel normalization: `log10().clamp(max-8).add(4)/4`
- Do NOT use tch's `stft_center` — it errors when `align_to_window` is set

### Audio Encoder Chunking (`audio_encoder.rs`)
- Chunk mel into n_window*2 (100) frame segments
- **Process ALL chunks as a single batch** through Conv2d (not one-at-a-time)
- Tail chunks zero-padded to chunk_size before Conv2d, then valid tokens extracted
- Block-diagonal windowed attention mask for chunk boundaries
- Output token count per chunk: apply `(len-1)//2 + 1` three times (one per Conv2d)

### MRoPE Position IDs (CRITICAL)
- All 3 MRoPE dimensions use **identical** sequential positions [0, 1, 2, ..., N-1]
- NO special handling for audio tokens — all dims are the same
- Wrong position IDs cause completely wrong transcription output
- Section sizes [24, 20, 20] = 64 total (half of head_dim=128), mapped contiguously by default
- cos/sin are doubled: `cos[j] == cos[j + half_dim]` for standard RoPE pattern

### Inference Loop (`inference.rs`)
- Prompt format: `<|im_start|>system\n<|im_end|>\n<|im_start|>user\n<|audio_start|>[AUDIO_PAD × N]<|audio_end|><|im_end|>\n<|im_start|>assistant\n`
- Audio embeddings injected via `slice_scatter` at AUDIO_PAD positions
- Prefill: full causal mask, populates KV cache for all layers
- Decode: single-token steps, **no mask needed** (causal mask for seq_len=1 is all-zeros)
- MRoPE cos/sin precomputed for all positions, indexed per step via `narrow`
- EOS tokens: ENDOFTEXT (151643) or IM_END (151645)

### Special Token IDs (`tokenizer.rs`)
| Token | ID |
|-------|-----|
| IM_START | 151644 |
| IM_END | 151645 |
| ENDOFTEXT | 151643 |
| AUDIO_START | 151669 |
| AUDIO_END | 151670 |
| AUDIO_PAD | 151676 |
| ASR_TEXT | 151704 |

## MLX Backend Notes

### Lazy Evaluation Model
MLX uses lazy evaluation: operations build a computation graph that executes only when `eval()` is called or data is read. Call `eval()` at outer loop boundaries, not per-layer.

Current eval placement:
- After audio encoder forward (materialize before decode phase)
- After prefill logits (materialize before decode loop)
- `int64_value()` / `item_i64()` implicitly call eval (forces graph execution for argmax)

### Fused Operations (Applied)
These replace multiple discrete ops with single Metal kernels:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [second-state/qwen3_asr_rs](https://github.com/second-state/qwen3_asr_rs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
