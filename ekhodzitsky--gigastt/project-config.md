---
trigger: always_on
description: enables token-bucket limiter on `/v1/*`; `/health` is exempt. Returns HTTP 429
---

# gigastt — Agent Guide

> Local speech-to-text server powered by GigaAM v3 e2e_rnnt. On-device Russian
> speech recognition via ONNX Runtime. No cloud APIs, no API keys, full privacy.
>
> Repository: https://github.com/ekhodzitsky/gigastt  
> crates.io: https://crates.io/crates/gigastt  
> License: MIT

## Project Overview

**gigastt** is a single-binary Rust server that turns any machine into a
real-time Russian speech-to-text endpoint. It loads the GigaAM v3 RNN-T model
(Conformer encoder + LSTM decoder + joiner, 240M params) via ONNX Runtime and
exposes:

- **WebSocket** (`/v1/ws`) — streaming transcription with partial/final results
- **REST** (`/v1/transcribe`) — file upload, full JSON response
- **SSE** (`/v1/transcribe/stream`) — file upload, streaming Server-Sent Events
- **CLI** — `serve`, `download`, `transcribe`, `quantize` commands

The model (~850 MB FP32, ~225 MB INT8) auto-downloads from HuggingFace on first
run. INT8 quantization is native Rust (no Python), always compiled since v0.9.0,
and auto-invoked on first `serve`/`download` unless `--skip-quantize` is passed.

### Key metrics

| Property | Value |
|---|---|
| WER (Russian) | **11.4%** (9 994 Golos crowd samples, 50 394 words, 95% CI [10.9%, 11.9%]) |
| Latency (16s audio, M1) | ~700 ms |
| Memory (RSS) | ~560 MB |
| Concurrent sessions | 4 (configurable via `--pool-size`) |

## Technology Stack

- **Language**: Rust 2024 edition, stable toolchain (1.88+)
- **ONNX Runtime**: `ort` 2.0.0-rc.12
- **Async runtime**: tokio (full features)
- **HTTP + WebSocket server**: axum 0.8 (`ws`, `multipart`)
- **CLI**: clap 4 (derive, env)
- **Serialization**: serde + serde_json
- **Logging**: tracing + tracing-subscriber (env-filter)
- **Error handling**: anyhow (internal), `GigasttError` (public API)
- **Audio decoding**: symphonia (AAC, MP3, OGG, FLAC, WAV, PCM)
- **Audio resampling**: rubato 0.16
- **FFT**: rustfft 6
- **Protobuf**: prost 0.14 + prost-build 0.14 (build-time)
- **Rate limiting**: in-tree token-bucket (dashmap-backed)
- **Metrics**: in-tree Prometheus text encoder (optional `--metrics` flag)

### Execution providers (compile-time features)

| Platform | Feature | Provider |
|---|---|---|
| macOS ARM64 | `--features coreml` | CoreML + Neural Engine |
| Linux x86_64 + NVIDIA | `--features cuda` | CUDA 12+ |
| Android / ARM64 | `--features nnapi` | NNAPI (via `ort/nnapi`) |
| Any | _(default)_ | CPU |

Features `coreml` and `cuda` are **mutually exclusive**. `nnapi` is not mutually exclusive with either.

## Build Requirements

- Rust 1.88+ (stable)
- `protoc` (Protocol Buffers compiler) on `PATH` — required by `build.rs` which
  regenerates ONNX protobuf types via `prost-build`
  - macOS: `brew install protobuf`
  - Debian/Ubuntu: `apt install protobuf-compiler`
- **Build-time network fetch:** `ort`'s default `download-binaries` feature downloads a
  prebuilt onnxruntime native library over the network at build time (outside `Cargo.lock`,
  verified by an embedded checksum). The "no cloud" guarantee is runtime-only. For
  air-gapped builds, use `ort` with `default-features = false` + `load-dynamic` and pin the
  native library via `ORT_*` env vars / `.cargo/config.toml`.

## Build Commands

```sh
# Debug build (CPU only, any platform)
cargo build

# Release build (LTO, stripped, single codegen unit)
cargo build --release

# macOS ARM64 with CoreML / Neural Engine
cargo build --release --features coreml

# Linux x86_64 with NVIDIA CUDA 12+
cargo build --release --features cuda

# Android with NNAPI
cargo build --release --features nnapi

# With speaker diarization support
cargo build --release --features diarization
```

## Test Commands

The project uses a three-tier test architecture:

### Unit tests (no model required, run in CI on every PR)

```sh
cargo test --workspace               # unit tests across the workspace
cargo clippy                         # Lint (zero warnings expected)
cargo fmt --check                    # Format check
```

Unit tests live in `#[cfg(test)] mod tests` at the bottom of each source file.
They use synthetic data. Test naming convention: `test_<what>_<expected_behavior>`.

### E2E tests (require model ~850 MB, run in CI on main push only)

```sh
# Download model first
cargo run -- download

# Run all e2e tests serially (single-threaded to avoid OOM)
cargo test -p gigastt --test e2e_rest --test e2e_ws --test e2e_errors --test e2e_shutdown --test e2e_rate_limit -- --ignored --test-threads=1
```

| Test file | Coverage |
|---|---|
| `tests/e2e_rest.rs` | REST API: health, transcribe, SSE streaming, error paths |
| `tests/e2e_ws.rs` | WebSocket: ready, audio, stop, configure, errors, concurrent |
| `tests/e2e_errors.rs` | Error paths: oversized body/frame, pool saturation, idle timeout |
| `tests/e2e_shutdown.rs` | Graceful shutdown: WS final + close, SSE termination, max-session cap |
| `tests/e2e_rate_limit.rs` | Per-IP rate limiter 429 behavior |

Shared helpers are in `tests/common/mod.rs` (server startup with shutdown handle,
WAV generation, WebSocket connect, readiness polling).

### Load & soak tests (require model, run locally + nightly CI)

```sh
cargo test -p gigastt --test load_test -- --ignored           # 3 load tests

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ekhodzitsky/gigastt](https://github.com/ekhodzitsky/gigastt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
