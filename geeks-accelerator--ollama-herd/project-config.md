---
trigger: always_on
description: > This file is the onboarding doc for AI coding agents. If you're a human, welcome — the same applies to you.
---

# AGENTS.md — Ollama Herd

> This file is the onboarding doc for AI coding agents. If you're a human, welcome — the same applies to you.

Ollama Herd is a smart inference router that turns a collection of devices running Ollama into a single local AI fleet. It routes LLM, embedding, image generation, and speech-to-text requests to the best available node using a 7-signal scoring engine, with zero config files and zero cloud dependency. The design philosophy is **node sovereignty** (each node works standalone; the router coordinates but never controls), **two-person scale** (two commands, no Docker, no Kubernetes), and **human-readable state** (JSONL logs, SQLite traces, JSON config — `grep` and `sqlite3` are your debuggers). v0.7.0 ships a native fastembed text embedding server that routes `nomic-embed-text` out of Ollama's inference queue entirely, plus a "Node Models" dashboard showing cards for all backends (Ollama, MLX, fastembed, vision embedding).

---

## Setup

**Requirements:** Python 3.11+, [uv](https://github.com/astral-sh/uv)

```bash
git clone https://github.com/geeks-accelerator/ollama-herd.git
cd ollama-herd
uv sync --all-extras          # core + all optional deps (recommended for development)
```

**Optional extras** (all included in `--all-extras`):

| Extra | What it adds | Command |
|-------|-------------|---------|
| `dev` | pytest, pytest-asyncio | `uv sync --extra dev` |
| `embedding` | onnxruntime, Pillow, numpy, huggingface-hub, fastembed | `uv sync --extra embedding` |

**macOS-only system deps** (not required for core routing):
- `uv tool install mflux` — FLUX image generation
- `uv tool install diffusionkit` — Stable Diffusion 3/3.5
- `pip install 'mlx-qwen3-asr[serve]'` — speech-to-text

---

## Running locally

```bash
uv run herd                                        # router on :11435
uv run herd-node                                   # node agent (mDNS auto-discovers router)
uv run herd-node --router-url http://localhost:11435  # explicit URL
```

**Dashboard:** `http://localhost:11435/dashboard`

**Config file:** `~/.fleet-manager/env` — auto-loaded at startup by both processes. All `FLEET_` vars go here. Shell env wins if both are set. Template: `docs/examples/fleet-env.example`.

**Logs:** `~/.fleet-manager/logs/herd.jsonl` (router) and `herd-node.jsonl` (node agent). Separated since 0.6.2 — scan both. Correct grep pattern: `'"level": "ERROR"'` (space after colon — `json.dumps` emits whitespace).

**Trace DB:** `~/.fleet-manager/latency.db` (SQLite, WAL mode).

**Port map:**

| Port | Service |
|------|---------|
| 11434 | Ollama (not ours) |
| 11435 | herd router |
| 11436 | image generation (mflux / DiffusionKit) |
| 11437 | speech-to-text |
| 11438 | vision embedding (ONNX) |
| 11439 | text embedding (fastembed) |
| 11440+ | MLX servers (one per `FLEET_NODE_MLX_SERVERS` entry) |

---

## Testing

```bash
uv sync --extra dev
uv run pytest                          # 1006 tests, ~40s
uv run pytest tests/test_server/       # server tests only
uv run pytest tests/test_node/         # node agent tests only
uv run pytest -k "test_scorer"         # single test module
uv run pytest --tb=short -q            # compact output
```

**No real Ollama required.** All tests are mocked. `tests/conftest.py` sets up in-memory SQLite, fake registries, and mock httpx transports — the full test suite runs offline with no network calls.

**Test layout:**

```
tests/
  conftest.py              shared fixtures (app, registry, trace store, mock httpx)
  test_server/             router: scoring, queues, streaming, health, routing, trace store
  test_node/               node agent: heartbeat, MLX supervisor, capacity, platform
  test_models/             Pydantic model validation
  test_common/             env file loading, shared utilities
```

The two known warnings (`coroutine '_dummy_process' was never awaited`) are pre-existing and do not indicate test failures.

---

## Code style

- **Ruff** for lint and format (`uv run ruff check src/` + `uv run ruff format src/`). Config in `pyproject.toml`.
- **Fully async** — no sync blocking calls anywhere. Every route handler, background task, and DB call must be `async`.
- **Pydantic v2** models for all data structures. No raw dicts crossing module boundaries.
- **`src/` layout** — package is `src/fleet_manager/`. Build backend is hatchling.
- **Line length:** 100 characters.
- No pre-commit hooks configured — run ruff manually before committing.

---

## Project layout

```
src/fleet_manager/
  server/
    app.py                 FastAPI app factory, lifespan, middleware
    scorer.py              7-signal routing scorer (thermal, memory, queue, wait, affinity, availability, context fit)
    queue_manager.py       per-node:model queues, dynamic concurrency, zombie reaper
    streaming.py           httpx proxy → Ollama, NDJSON↔SSE, auto-retry, context protection
    health_engine.py       36 automated health checks — all wired to /dashboard/api/health
    registry.py            in-memory node registry, heartbeat ingestion, node state
    context_optimizer.py   dynamic num_ctx: analyzes token usage, queues Ollama restarts

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [geeks-accelerator/ollama-herd](https://github.com/geeks-accelerator/ollama-herd) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-24 -->
