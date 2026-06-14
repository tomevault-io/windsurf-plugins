---
trigger: always_on
description: Rust rewrite of `vllm bench serve` — a high-performance benchmark client for vLLM serving endpoints. Standalone binary, no Python dependency at runtime.
---

# AGENTS.md

## Project Overview

Rust rewrite of `vllm bench serve` — a high-performance benchmark client for vLLM serving endpoints. Standalone binary, no Python dependency at runtime.

## Build & Test

```bash
# Build release binary
cargo build --release

# Run all tests
cargo test

# Run ignored integration tests (requires network for tokenizer download)
cargo test -- --ignored
```

## Architecture

- `src/main.rs` — Entry point, mimalloc, tokio runtime, mode dispatch (compare/sweep/multi-run/multi-turn/single)
- `src/cli.rs` — clap derive CLI args (~50+ flags)
- `src/config.rs` — Validated config from CLI; `GoodputConfig`, `RampUpConfig`, sampling param merging
- `src/error.rs` — `BenchError` enum (Http, Json, Tokenizer, Config, EndpointTimeout, Backend, Io)
- `src/benchmark.rs` — Core benchmark orchestrator (spawn-per-request with tokio + Semaphore; fetches speculative decoding metrics from `/metrics`)
- `src/multi_turn.rs` — Multi-turn conversation orchestrator (channel-based worker pool, sequential turns per conversation)
- `src/sweep.rs` — Concurrency/rate parameter sweep (`--sweep-max-concurrency`, `--sweep-request-rate`)
- `src/multi_run.rs` — N-run aggregation with mean/std/min/max/CV (`--num-runs`)
- `src/compare.rs` — Side-by-side diff of two result JSON files (`--compare`)
- `src/tokenizer.rs` — `TokenizerKind` enum: Local(HuggingFace), Tiktoken, OR Server-side `/tokenize`+`/detokenize` fallback
- `src/tiktoken.rs` — Tiktoken BPE loader (`.tiktoken`/`.model` files; built-in encodings o200k_base/cl100k_base; pat_str extraction from Python source)
- `src/rate_control.rs` — Gamma/Poisson request scheduling + linear/exponential ramp-up
- `src/ready_checker.rs` — Endpoint readiness with retry
- `src/backends/` — Backend implementations (enum dispatch, not trait objects)
  - `mod.rs` — `Backend` enum, `RequestFuncInput`/`RequestFuncOutput` (includes `messages` field for multi-turn)
  - `streaming.rs` — SSE parser (`StreamedResponseHandler`) with speculative JSON parse for split TCP segments
  - `openai_completions.rs` — `/v1/completions` backend
  - `openai_chat.rs` — `/v1/chat/completions` backend (uses `input.messages` when set; zero-copy raw JSON payload for multimodal)
  - `pooling.rs` — Non-streaming pooling/embedding backends: `openai-embeddings`, `openai-embeddings-chat`, `vllm-pooling`, `vllm-rerank`
- `src/datasets/random.rs` — Random dataset generation with rayon parallelism
- `src/datasets/random_mm.rs` — Random multimodal dataset (synthetic JPEG images, bucket config sampling, pre-serialized JSON fragments)
- `src/datasets/sharegpt.rs` — ShareGPT JSON loader + HuggingFace Hub auto-download with caching
- `src/datasets/sonnet.rs` — Sonnet dataset (built-in Shakespeare sonnets via `include_str!("../../sonnet.txt")`; controllable token length + shared prefix; mirrors Python `SonnetDataset`)
- `src/datasets/speed_bench.rs` — NVIDIA SPEED-Bench loader (HF datasets-server API, 6 configs, 11 categories, local cache)
- `src/datasets/hf_dataset.rs` — Generic HuggingFace dataset loader (datasets-server API, column auto-detection)
- `src/datasets/multi_turn.rs` — Multi-turn synthetic generator + ShareGPT multi-turn loader (3-tier prefix sharing: global/conversation/unique-suffix; `per_turn_input_len`)
- `src/metrics/mod.rs` — `BenchmarkMetrics` and `MultiTurnMetrics` structs
- `src/metrics/calculator.rs` — TTFT/TPOT/ITL/E2EL/throughput stats, goodput SLO checking, peak concurrency, `calculate_multi_turn_metrics`
- `src/metrics/steady_state.rs` — Steady-state window detection (in-flight concurrency plateau via two-pointer start/end merge) + plateau throughput/TTFT/TPOT; gated on `--max-concurrency` set + `--request-rate inf` (closed-loop)
- `src/output/console.rs` — Terminal output matching Python format + multi-turn per-turn breakdown
- `src/output/json.rs` — JSON result file (compatible with Python schema) + multi-turn JSON with `per_turn_metrics`

## Key Design Decisions

- **Enum dispatch** for backends (avoids async trait object issues with `dyn`)
- **reqwest http1_only()** to match Python aiohttp behavior
- **rayon** for parallel dataset generation (key perf win over Python)
- **mimalloc** global allocator to reduce contention at 1400+ concurrency (page-agnostic; works on aarch64 64K-page kernels where jemalloc aborts with `LG_PAGE=12` builds)
- **Arc\<str\> prompts** zero-copy sharing across tokio tasks (~3GB savings at 100k prompts with 8k-token inputs)
- **Spawn-per-request** `tokio::spawn` + `Semaphore` (matches Python asyncio pattern)
- **Speculative JSON parse** in SSE handler — detects complete JSON before `\n\n` arrives, improving TTFT/ITL accuracy when TCP segments split
- **Tokenizer fallback chain**: Local HF → Tiktoken (`.tiktoken`/`.model` + built-in encodings) → Server-side `/tokenize`+`/detokenize`. Blocking HTTP in rayon threads for server fallback.
- **hf-hub** for downloading tokenizers and datasets from HuggingFace Hub
- **Pre-serialized mm fragments** (`Arc<str>`) for multimodal: image content stored as JSON strings, zero-copy concatenated into payload — avoids deep-cloning ~200KB+ base64 per request

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vllm-project/vllm-bench](https://github.com/vllm-project/vllm-bench) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-14 -->
