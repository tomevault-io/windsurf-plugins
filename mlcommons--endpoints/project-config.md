---
trigger: always_on
description: Guidelines for AI coding agents working in the MLPerf Inference Endpoint Benchmarking System repository.
---

# AGENTS.md

Guidelines for AI coding agents working in the MLPerf Inference Endpoint Benchmarking System repository.

## Project Overview

High-performance benchmarking tool for LLM inference endpoints targeting 50k+ QPS. Python 3.12+, Apache 2.0 licensed.

## Common Commands

```bash
# Development setup
uv sync --extra dev --extra test
uv run pre-commit install

# Testing
uv run pytest                                        # All tests (excludes slow/performance)
uv run pytest -m unit                                # Unit tests only
uv run pytest -m integration                         # Integration tests only
uv run pytest --cov=src --cov-report=html            # With coverage
uv run pytest -xvs tests/unit/path/to/test_file.py  # Single test file

# Code quality (run before commits)
uv run pre-commit run --all-files

# Local testing with echo server
uv run python -m inference_endpoint.testing.echo_server --port 8765
uv run inference-endpoint probe --endpoints http://localhost:8765 --model test-model

# CLI usage
uv run inference-endpoint benchmark offline --endpoints URL --model NAME --dataset PATH
uv run inference-endpoint benchmark online --endpoints URL --model NAME --dataset PATH --load-pattern poisson --target-qps 100
uv run inference-endpoint benchmark from-config --config config.yaml
```

### Backward-compatible setup (pip + venv)

Does not use `uv.lock` — dependency versions may differ from the lockfile.

```bash
python3.12 -m venv venv && source venv/bin/activate
pip install -e ".[dev,test]"
pre-commit install

# After activating the venv, commands run without the `uv run` prefix:
pytest                                        # All tests (excludes slow/performance)
pytest -m unit                                # Unit tests only
pytest -m integration                         # Integration tests only
pytest --cov=src --cov-report=html            # With coverage
pytest -xvs tests/unit/path/to/test_file.py  # Single test file

# Code quality — MUST run before every commit, no exceptions
pre-commit run --all-files

# Local testing with echo server
python -m inference_endpoint.testing.echo_server --port 8765
inference-endpoint probe --endpoints http://localhost:8765 --model test-model

# CLI usage
inference-endpoint benchmark offline --endpoints URL --model NAME --dataset PATH
inference-endpoint benchmark online --endpoints URL --model NAME --dataset PATH --load-pattern poisson --target-qps 100
inference-endpoint benchmark from-config --config config.yaml
```

## Architecture

### Core Data Flow

```
Dataset Manager --> Load Generator --> Endpoint Client --> External Endpoint
                        |
                   Metrics Collector (EventRecorder + MetricsReporter)
```

### Key Components

| Component           | Location                                                      | Purpose                                                                                                                                     |
| ------------------- | ------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------- |
| **Load Generator**  | `src/inference_endpoint/load_generator/`                      | Central orchestrator: `BenchmarkSession` owns the lifecycle, `Scheduler` controls timing, `LoadGenerator` issues queries                    |
| **Endpoint Client** | `src/inference_endpoint/endpoint_client/`                     | Multi-process HTTP workers communicating via ZMQ IPC. `HTTPEndpointClient` is the main entry point                                          |
| **Dataset Manager** | `src/inference_endpoint/dataset_manager/`                     | Loads JSONL, HuggingFace, CSV, JSON, Parquet datasets. `Dataset` base class with `load_sample()`/`num_samples()` interface                  |
| **Metrics**         | `src/inference_endpoint/metrics/`                             | `EventRecorder` writes to SQLite, `MetricsReporter` reads and aggregates (QPS, latency, TTFT, TPOT)                                         |
| **Config**          | `src/inference_endpoint/config/`, `endpoint_client/config.py` | Pydantic-based YAML schema (`schema.py`), `HTTPClientConfig` (single Pydantic model for CLI/YAML/runtime), `RuntimeSettings`                |
| **CLI**             | `src/inference_endpoint/main.py`, `commands/benchmark/cli.py` | cyclopts-based, auto-generated from `schema.py` and `HTTPClientConfig` Pydantic models. Flat shorthands via `cyclopts.Parameter(alias=...)` |
| **Async Utils**     | `src/inference_endpoint/async_utils/`                         | `LoopManager` (uvloop + eager_task_factory), ZMQ transport layer, event publisher                                                           |
| **OpenAI/SGLang**   | `src/inference_endpoint/openai/`, `sglang/`                   | Protocol adapters and response accumulators for different API formats                                                                       |

### Hot-Path Architecture

Multi-process, event-loop design optimized for throughput:

- `BenchmarkSession` thread schedules samples with busy-wait timing

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mlcommons/endpoints](https://github.com/mlcommons/endpoints) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
