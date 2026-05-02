---
trigger: always_on
description: Qwen3 TTS Rust (`qwen3-tts-rs`) is a pure-Rust inference engine for the [Qwen3 TTS](https://huggingface.co/Qwen/Qwen3-TTS-12Hz-0.6B-CustomVoice) text-to-speech model family. It converts text to natural-sounding speech using transformer-based language models that generate codec tokens, which are then decoded to audio by a BigVGAN vocoder.
---

# CLAUDE.md — Qwen3 TTS Rust Project Guide

## Project Purpose

Qwen3 TTS Rust (`qwen3-tts-rs`) is a pure-Rust inference engine for the [Qwen3 TTS](https://huggingface.co/Qwen/Qwen3-TTS-12Hz-0.6B-CustomVoice) text-to-speech model family. It converts text to natural-sounding speech using transformer-based language models that generate codec tokens, which are then decoded to audio by a BigVGAN vocoder.

The crate supports two computation backends:

- **tch** (default) — PyTorch/libtorch via `tch-rs` bindings. Works on Linux x86_64, Linux ARM64, and macOS.
- **mlx** — Apple MLX via C FFI. macOS Apple Silicon only, uses Metal GPU acceleration.

## Supported Models

| Model | Capability | Notes |
|-------|-----------|-------|
| `Qwen3-TTS-12Hz-0.6B-CustomVoice` | Preset speaker voices | 9 built-in speakers |
| `Qwen3-TTS-12Hz-0.6B-Base` | Voice cloning (ICL) | Has speaker encoder weights |
| `Qwen3-TTS-12Hz-1.7B-CustomVoice` | Preset voices + voice instruction control | Supports style instructions like "Speak urgently" |

Available speakers: `serena`, `vivian`, `uncle_fu`, `ryan`, `aiden`, `ono_anna`, `sohee`, `eric`, `dylan`.

## Build & Run

```bash
# tch backend (default) — requires libtorch or PyTorch
export LIBTORCH_USE_PYTORCH=1
cargo build --release

# MLX backend — requires git submodule
git submodule update --init --recursive
cargo build --release --no-default-features --features mlx

# Generate speech
cargo run --release --bin tts -- models/Qwen3-TTS-12Hz-0.6B-CustomVoice "Hello world" Vivian english

# Voice cloning (ICL)
cargo run --release --bin voice_clone -- models/Qwen3-TTS-12Hz-0.6B-Base reference.wav "Text to say" english "Reference transcript"

# API server (OpenAI-compatible)
cargo run --release --bin api_server -- models/Qwen3-TTS-12Hz-0.6B-CustomVoice --port 8080
```

## Architecture

The crate follows a backend-agnostic design. All neural network code (layers, inference, vocoder) operates on a unified `Tensor` type that dispatches to tch or MLX at compile time via `#[cfg(feature)]` gates. The two features (`tch-backend` and `mlx`) are mutually exclusive.

### Inference Pipeline

```
Text → Tokenizer → Transformer (autoregressive codec token generation)
                          ↓
                    Codec tokens → BigVGAN Vocoder → Waveform (24kHz f32 PCM)
```

For voice cloning (ICL mode), the pipeline adds a reference audio encoding step:

```
Reference audio → AudioEncoder → Codec tokens ─┐
Reference transcript ──────────────────────────┤
Target text ───────────────────────────────────┤
                                               ↓
                              Transformer (conditioned on reference) → Vocoder → Audio
```

---

## File Descriptions

### Root

| File | Description |
|------|-------------|
| `Cargo.toml` | Package manifest. Defines `tch-backend` (default) and `mlx` features as mutually exclusive. |
| `build.rs` | Build script. When `mlx` feature is active, builds the mlx-c git submodule via CMake and links Metal/Accelerate/MPS frameworks. No-op for tch backend. |
| `install.sh` | Installation helper script. |

### `src/lib.rs`

Library root. Declares all public modules, enforces mutual exclusivity of backends via `compile_error!`, and re-exports main types (`Qwen3TTSModel`, `Language`, `Speaker`, `AudioInput`, etc.).

### `src/tensor.rs`

Unified tensor API — the main backend abstraction layer. Wraps either `tch::Tensor` or `MlxArray` behind cfg gates. Exposes a common interface for creation, arithmetic, indexing, reshaping, and extraction that all model code uses.

### `src/config.rs`

Configuration types deserialized from model JSON files (`config.json`, `generation_config.json`). Includes `Qwen3TTSConfig` (model architecture params), `GenerationConfig` (sampling params), `SpeakerEncoderConfig`, `VocoderConfig`, and `TokenizerType` detection.

### `src/types.rs`

Core domain types: `Language` (English/Chinese/Japanese/Korean/Auto), `Speaker`, `VoiceInstruction`, `AudioInput` (FilePath/Url/Base64), `GenerationOutput`, `DecodedAudio`, `EncodedAudio`, `VoiceClonePromptItem`.

### `src/error.rs`

Error types using `thiserror`. `Qwen3TTSError` covers model loading, tokenization, inference, audio processing, and I/O failures. Exports a `Result<T>` type alias.

### `src/tokenizer.rs`

Wraps HuggingFace `tokenizers` crate. Loads `tokenizer.json`, handles special token IDs (`<|audio_bos|>`, `<|audio_eos|>`, `<|text_eos|>`, codec tokens), and formats chat-template prompts for different generation modes.

### `src/model.rs`

High-level model API (`Qwen3TTSModel`). Provides `generate_custom_voice()`, `generate_voice_clone()`, and `generate_voice_design()` methods. Also contains `GenerationParams` builder and text formatting logic.

### `src/inference.rs`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [second-state/qwen3_tts_rs](https://github.com/second-state/qwen3_tts_rs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
