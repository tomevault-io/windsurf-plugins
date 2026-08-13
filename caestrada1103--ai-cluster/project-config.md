---
trigger: always_on
description: This file provides guidance to AI coding agents (Claude Code, Cursor, Copilot, etc.) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to AI coding agents (Claude Code, Cursor, Copilot, etc.) when working with code in this repository.

## Project Overview

AICluster is a distributed LLM inference platform with two core components:
- **Coordinator** (`coordinator/`) — Python FastAPI service providing an OpenAI-compatible REST API, worker discovery, load balancing, and model registry.
- **Worker** (`worker/`) — Rust service that runs GPU inference through two engines and exposes a gRPC endpoint: **llama.cpp/GGUF** (primary/recommended engine for consumer GPUs — native quantization, e.g. Q4_K_M/Q5_K_M/Q8_0; opt-in via the `llamacpp` cargo feature) and **Burn** (FP32 reference/experimental engine, safetensors; `wgpu` is the **default** cargo build feature — llama.cpp is not compiled unless requested).

Clients talk REST to the coordinator; the coordinator talks gRPC (protobuf) to workers. Protocol definitions live in `proto/cluster.proto` and generated bindings are in `coordinator/proto/` and built by `worker/build.rs`.

## Commands

### Docker Compose (full stack)
```bash
docker compose up -d --build   # Start coordinator + worker + Prometheus + Grafana + Open-WebUI
docker compose logs -f
docker compose down
```

### Coordinator (Python)
```bash
# Run from the REPO ROOT — `cd coordinator && uvicorn main:app` breaks package imports.
pip install -r coordinator/requirements-dev.txt   # runtime+lint+test (runtime only: requirements.txt)
uvicorn coordinator.main:app --reload --host 127.0.0.1 --port 8000
```
> Security note: the coordinator refuses to start with a non-loopback `--host`
> (e.g. `0.0.0.0`) unless `COORDINATOR_API_KEYS` is set — secure by default.
> For a LAN-reachable coordinator, set `COORDINATOR_API_KEYS` first
> (comma-separated secrets, e.g. `openssl rand -hex 32`) and pass
> `--host 0.0.0.0`. See `.env.example` and `docs/deployment.md`.

### Worker (Rust)
`wgpu` (Burn engine, FP32) is the **default** cargo feature; add `llamacpp` for the **recommended** GGUF/quantized-inference engine on consumer GPUs.
```bash
cd worker
cargo build --release --features wgpu    # Universal — Vulkan/DX12/Metal, auto-detects AMD/NVIDIA/Intel (default; Burn engine, FP32)
cargo build --release --features cuda    # NVIDIA base kernels (runtime backend type is still Wgpu — native wiring planned)
cargo build --release --features rocm    # AMD base kernels (runtime backend type is still Wgpu — native wiring planned)
cargo build --release --features metal   # macOS — Metal via wgpu
# llama.cpp engine (GGUF models, recommended for consumer-GPU quantized inference) — combine with a Burn backend feature:
cargo build --release --features wgpu,llamacpp                  # llama.cpp on CPU
cargo build --release --features wgpu,llamacpp,llamacpp-vulkan  # llama.cpp Vulkan offload
cargo build --release --features cuda,llamacpp,llamacpp-cuda    # llama.cpp CUDA offload
# Cross-node ggml-RPC roles (peer/lead), opt-in — no extra deps, spawns external binaries:
cargo build --release --features wgpu,llamacpp-rpc
./target/release/ai-worker --port 50051
```
> There is no CPU-only/ndarray build; a GPU (or Vulkan software rasterizer) is required.

### Tests
```bash
# Rust worker tests (must pass a backend feature — CI uses wgpu)
cd worker && cargo test --features wgpu
cargo test --features wgpu config::tests::test_name   # single test by path
# llama.cpp engine (compiles llama.cpp via cmake; needs cmake + libclang)
cargo check --features llamacpp
cargo test --features llamacpp llamacpp_engine                  # unit tests
cargo test --features llamacpp -- --ignored llamacpp            # e2e (network, ~1 MiB GGUF)
# Cross-node ggml-RPC roles (no extra deps — spawns external binaries, no GPU needed to test)
cargo test --features wgpu,llamacpp-rpc rpc_server

# Python unit tests (run from repo root)
pytest coordinator/
pytest coordinator/tests/test_router.py                # single file
pytest coordinator/tests/test_router.py::test_name -v  # single test

# Integration / client smoke tests (require a running coordinator + worker)
python tests/test_client.py
python tests/cluster_chat.py
```
> Note: `coordinator/tests/` contains the unit suite covering `models`, `config`, `router`, and coordinator error paths (run `pytest coordinator/ -q` for the current count). Rust unit tests live inline (`worker/src/config.rs`, `worker/src/gpu_manager.rs`, `worker/models/common.rs`).

### Linting
```bash
# Python: Black (line-length 100) + Ruff + MyPy strict
black --line-length 100 coordinator/
ruff check coordinator/
mypy coordinator/

# Rust (both enforced in CI)
cargo fmt -- --check
cargo clippy -p ai-worker --features wgpu -- -D warnings
```

## Architecture

```
Client (REST) → Coordinator (FastAPI) → Workers (Rust: llama.cpp + Burn) → GPU
                      │
              Prometheus / Grafana
```

**Coordinator modules** (`coordinator/`):
- `main.py` — FastAPI app entry point, lifespan, CORS, Prometheus ASGI mount
- `api.py` — FastAPI routes (`/v1/completions`, `/v1/chat/completions`, `/v1/models`, `/v1/workers`, `/health`, `/metrics`)
- `coordinator.py` — Core orchestration logic

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [caestrada1103/ai-cluster](https://github.com/caestrada1103/ai-cluster) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
