---
trigger: always_on
description: Rust rewrite of `vllm bench serve` — a high-performance benchmark client for vLLM serving endpoints. Standalone binary, no Python dependency at runtime.
---

# AGENTS.md

## Project Overview

Rust rewrite of `vllm bench serve` — a high-performance benchmark client for vLLM serving endpoints. Standalone binary, no Python dependency at runtime.

Member crate `vllm-bench` of the `rust/` workspace. Uses workspace dependencies and lints; the workspace `[profile.release]` (thin LTO, `panic = "abort"`) applies. Note the workspace bans rustls/ring (`rust/deny.toml`) — all HTTP must stay on native-tls, which is why HF Hub downloads go through `src/hub.rs` (hf-hub's async reqwest/native-tls API) instead of hf-hub's ureq backend.

## Build & Test

Run from the `rust/` workspace root:

```bash
# Build release binary (rust/target/release/vllm-bench)
cargo build -p vllm-bench --release

# Run all tests
cargo test -p vllm-bench

# Run ignored integration tests (requires network for tokenizer download)
cargo test -p vllm-bench -- --ignored
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
- `src/hub.rs` — `HubRepo`: thin wrapper over hf-hub's async (reqwest/native-tls) API — file download and repo listing, optional revision; the sync ureq backend is unusable here because it pulls rustls, which `rust/deny.toml` bans
- `src/rate_control.rs` — Gamma/Poisson request scheduling + linear/exponential ramp-up
- `src/ready_checker.rs` — Endpoint readiness with retry
- `src/backends/` — Backend implementations (enum dispatch, not trait objects)
    - `mod.rs` — `Backend` enum, `RequestFuncInput`/`RequestFuncOutput` (includes `messages` field for multi-turn)
    - `streaming.rs` — SSE parser (`StreamedResponseHandler`) with speculative JSON parse for split TCP segments
    - `openai_completions.rs` — `/v1/completions` backend
    - `openai_chat.rs` — `/v1/chat/completions` backend (uses `input.messages` when set; zero-copy raw JSON payload for multimodal)
    - `pooling.rs` — Non-streaming pooling/embedding backends: `openai-embeddings`, `openai-embeddings-chat`, `vllm-pooling`, `vllm-rerank`
- `src/datasets/random.rs` — Random dataset generation with rayon parallelism
- `src/datasets/random_mm.rs` — Random multimodal dataset (synthetic JPEG images, bucket config sampling, pre-serialized JSON fragments); `--enable-multimodal-chat` pre-builds the chat `messages` array at dataset time (mirrors Python's `apply_multimodal_chat_transformation`)
- `src/datasets/sharegpt.rs` — ShareGPT JSON loader + HuggingFace Hub auto-download with caching
- `src/datasets/sonnet.rs` and `src/datasets/sonnet.txt` — Sonnet dataset (built-in Shakespeare sonnets via `include_str!("sonnet.txt")`; controllable token length + shared prefix; mirrors Python `SonnetDataset`)
- `src/datasets/speed_bench.rs` — NVIDIA SPEED-Bench loader (HF datasets-server API, 6 configs, 11 categories, local cache)
- `src/datasets/hf_dataset.rs` — Generic HuggingFace dataset loader (parquet shards via hf-hub into the standard HF hub cache; datasets-server for config/split discovery; native main-branch parquet/JSON fallback for private datasets; column auto-detection)
- `src/datasets/custom.rs` — Custom JSONL dataset (`{"prompt": ..., "output_tokens": ...}` per line; `--custom-output-len -1` uses per-line output_tokens; prompts always sent raw — no client-side chat template)
- `src/datasets/prefix_repetition.rs` — Prefix repetition dataset (N shared prefixes × fresh random suffixes, standard prefix-cache stress; mirrors Python `PrefixRepetitionRandomDataset`)
- `src/datasets/random_rerank.rs` — Random rerank dataset (one query + batched documents per request for `vllm-rerank`; `--no-reranker` for embedding-based scoring; mirrors Python `RandomDatasetForReranking`)
- `src/datasets/multi_turn.rs` — Multi-turn synthetic generator + ShareGPT multi-turn loader (3-tier prefix sharing: global/conversation/unique-suffix; `per_turn_input_len`)
- `src/metrics/mod.rs` — `BenchmarkMetrics` and `MultiTurnMetrics` structs
- `src/metrics/calculator.rs` — TTFT/TPOT/ITL/E2EL/throughput stats, goodput SLO checking, peak concurrency, `calculate_multi_turn_metrics`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [weicj/vLLM-2080Ti-Definitive](https://github.com/weicj/vLLM-2080Ti-Definitive) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
