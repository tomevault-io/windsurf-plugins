---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

LIFT (Loaded Impact on Final Task) is an evaluation framework for **agent self-evolution**. It does not measure an agent's out-of-the-box ability — it measures whether an agent actually improves, and by how much, on holdout tasks after completing warmup tasks and triggering one evolve step.

**Core paradigm**: every holdout task is run twice — once with a clean baseline image, and once with a delta image committed after warmup + evolve. The diff measures improvement.

Agents are hosted in Docker containers (OpenClaw and other runtimes); the pipeline is a single-process asyncio orchestrator. Work and judge agents now run in separate sibling containers with the same image/workspace/load state. Holdout container fan-out is `max_parallel_suites × per-cell task parallelism × 2 phases × 2 roles(work+judge)`; with defaults (3 parallel cells, moderate holdout task counts) peak is commonly dozens of concurrent containers, and can climb into the hundreds when `--max-parallel-suites` is raised.

## Common Commands

> **Python 环境约定**：所有 `python` / `pytest` 命令默认在 `lift` conda 环境执行。非交互调用直接用绝对路径 `/root/miniconda3/envs/lift/bin/python`；交互终端可先 `conda activate lift`。系统 `/usr/bin/python` **未安装项目依赖**（缺 pydantic 等），会立即报错。

```bash
# Build the OpenClaw evaluation image (rebuild required after runtime changes)
# Default builds the base image; pass --with-evolve to include the evolution plugin
bash agent-runtimes/openclaw/build-image.sh                # → lift-openclaw-base:latest
bash agent-runtimes/openclaw/build-image.sh --with-evolve  # → lift-openclaw-with-evolve:latest
# OpenSpace MCP plugin (quality-first skill hub); mutually exclusive with --with-evolve (pick one)
bash agent-runtimes/openclaw/build-image.sh --with-openspace                # → lift-openclaw-with-openspace:latest
# agentmemory memory plugin; mutually exclusive with --with-evolve / --with-openspace
bash agent-runtimes/openclaw/build-image.sh --with-agentmemory              # → lift-openclaw-with-agentmemory:latest
# Hermes OpenSpace variant
bash agent-runtimes/hermes/build-image.sh --with-openspace  # → lift-hermes-with-openspace:latest
# Hermes / OpenHuman agentmemory variants
bash agent-runtimes/hermes/build-image.sh --with-agentmemory    # → lift-hermes-with-agentmemory:latest
bash agent-runtimes/openhuman/build-image.sh --with-agentmemory # → lift-openhuman-with-agentmemory:latest

# One-shot: build all runtime images (SSoT for the full command list: docs/build-images.md)
bash scripts/build-all-images.sh                            # build everything
bash scripts/build-all-images.sh --only openclaw-base,hermes  # subset
bash scripts/build-all-images.sh --list                     # list target names

# ByteDance intranet build (defaults go through public mirrors; switch via env vars)
APT_MIRROR=http://mirrors.byted.org \
PIP_INDEX_URL=https://bytedpypi.byted.org/simple/ \
bash agent-runtimes/openclaw/build-image.sh --with-evolve

# Smoke test (warmup + delta only, skips holdout)
python -m src.cli.lift_main -r openclaw \
  --benchmark_dir assets/benchmarks_demo --suite hello.json \
  --run-id smoke-test --warmup-only

# Full LIFT evaluation (with terminal TUI + browser dashboard)
python -m src.cli.lift_main -r openclaw \
  --benchmark_dir assets/benchmarks_demo --suite hello.json \
  --run-id my-run --tui --dashboard 8080

# Post-process only (rebuild dashboard / metrics from existing report.json)
python -m src.cli.lift_main -r openclaw --evaluate-only --run-id my-run

# Pull benchmark markdowns from TOS / HuggingFace / ModelScope and convert to suite JSON
python -m src.cli.preprocess

# Unit tests
python -m pytest src/lift/tests -q
```

## Architecture

### Three-Layer Design

```
CLI / Pipeline (src/cli, src/lift/pipeline)
    ↓ orchestrates repeat × suite × phase, writes report, mounts status dashboard
AgentRuntimeAdapter (src/lift/adapters)
    ↓ runtime-specific: container lifecycle, evolve hooks, chat interface, delta materialization
lift/eval (src/lift/eval)
    ↓ evaluation kernel: per-task work↔judge multi-turn loop (runtime-agnostic)
```

### Key Components

| Component | Path | Purpose |
|-----------|------|---------|
| CLI entry | `src/cli/lift_main.py` | Argument parsing; dispatches full run vs. `--evaluate-only` |
| Pipeline | `src/lift/pipeline/lift_pipeline.py` | Flattens repeat × suite into cells; cell-level concurrency + failure retry |
| Adapter base | `src/lift/adapters/base.py` | `AgentRuntimeAdapter` template: `produce_delta` / `run_before_load` / `run_after_load` |
| Container adapter | `src/lift/adapters/container/` | Generic Docker lifecycle, `docker commit` to materialize delta |
| OpenClaw base | `src/lift/adapters/openclaw/` | OpenClaw container, chat, `agents add --model`, trajectory.jsonl parsing |
| OpenClaw with evolve | `src/lift/adapters/openclaw_with_evolve/` | Runs `openclaw learn review` after warmup, plus signal pipeline |
| Group memory mixin | `src/lift/adapters/group_memory/` + `openclaw_multi_user/` | Multi-container warmup, evolve writes to external memory service, delta reuses base image |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [FeiZhuNiU-INFJA/LIFT](https://github.com/FeiZhuNiU-INFJA/LIFT) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-21 -->
