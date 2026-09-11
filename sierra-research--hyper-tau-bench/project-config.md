---
trigger: always_on
description: > Instructions for AI coding agents working on the τ^τ-bench (Hyper-τ) codebase.
---

# AGENTS.md

> Instructions for AI coding agents working on the τ^τ-bench (Hyper-τ) codebase.

## Project Overview

τ^τ-bench (engineering name: Hyper-τ) evaluates how well an LLM Developer can build a working customer-service agent from realistic evidence, scoring the built agent with a τ³-bench inner loop. This repository contains both layers: the outer construction loop (`src/tau2/hyper/`) and the full τ³-bench simulation framework it wraps — text and voice interactions in half-duplex (turn-based) and full-duplex (simultaneous/streaming) communication modes. Domains include `mock`, `airline_plus`, `retail_plus`, `telecom`, and `banking_knowledge`.

## Hyper-τ Domain Policy

For all Hyper-τ work, `airline_plus` and `retail_plus` are the maintained
airline and retail corpora. The canonical `airline` and `retail` Hyper-τ
evidence trees were frozen legacy memorization baselines and are not part of
this release tree (the canonical domains under `src/tau2/domains/` and
`data/tau2/domains/` remain — the plus domains subclass them).

- If a Hyper-τ task, script, document, or command points at canonical
  `airline` or `retail`, redirect it to `airline_plus` or `retail_plus`.
- The canonical `airline`/`retail` domains are deprecated, frozen legacy
  baselines kept only for an explicit future migration or ablation study
  (`tau2 hyper-tau --allow-legacy-domain`); do not backport routine
  `retail_plus` or `airline_plus` fixes into them.

## Setup

```bash
uv sync           # create venv, install deps from lockfile
uv run tau2 check-data  # verify installation
```

Environment variables: copy `.env.example` to `.env` and set API keys. Uses [LiteLLM](https://github.com/BerriAI/litellm) for LLM provider abstraction.

Required keys depend on the task:
- `OPENAI_API_KEY` / `OPENROUTER_API_KEY` — every model seat in the shipped configuration; `model_routing.toml` at the repo root decides which key and endpoint serve each model id (`ANTHROPIC_API_KEY` only for the `claude-code` Developer harness)
- `ELEVENLABS_API_KEY` — voice synthesis
- `DEEPGRAM_API_KEY` — voice transcription

## Common Commands

| Command | What it does | Required install |
|---------|-------------|-----------------|
| `make test` | Run core tests (skips voice, streaming, gym, banking_knowledge) | `uv sync --extra dev` |
| `make test-voice` | Run voice + streaming tests | `uv sync --extra voice --extra dev` |
| `make test-knowledge` | Run banking_knowledge tests | `uv sync --extra knowledge --extra dev` |
| `make test-gym` | Run gymnasium tests | `uv sync --extra gym --extra dev` |
| `make test-all` | Run all tests | `uv sync --all-extras` |
| `make lint` | Lint with ruff | `uv sync --extra dev` |
| `make format` | Format with ruff | `uv sync --extra dev` |
| `make lint-fix` | Lint and auto-fix | `uv sync --extra dev` |
| `make check-all` | Run lint + format (same as pre-commit hook) | `uv sync --extra dev` |
| `make clean` | Remove venv, caches, build artifacts | — |
| `make env-cli` | Interactive environment CLI for testing domain tools | — |

`make test` is the safe default -- it works with just `uv sync --extra dev` and does not require voice, knowledge, or gym packages. Always run `make check-all` before committing. A pre-commit hook enforces this.

## Running Evaluations

```bash
# Text half-duplex (standard)
tau2 run --domain airline --agent-llm gpt-4.1 --user-llm gpt-4.1 --num-trials 1 --num-tasks 5

# Voice full-duplex (audio native)
tau2 run --domain retail --audio-native --num-tasks 1 --verbose-logs

# Knowledge domain (requires --retrieval-config)
tau2 run --domain banking_knowledge --retrieval-config qwen_embeddings --agent-llm gpt-4.1 --user-llm gpt-4.1 --num-tasks 5
```

Results go to `data/simulations/`. Use `tau2 view` to browse them.

## Architecture

```
src/tau2/
├── agent/           # Agent implementations (half-duplex and full-duplex)
├── api_service/     # FastAPI-based API service
├── config.py        # Central configuration (single source of truth for defaults)
├── cli.py           # CLI entry point (tau2 command)
├── data_model/      # Pydantic data models (messages, trajectories, etc.)
├── domains/         # Domain definitions (airline, mock, retail, telecom, banking_knowledge)
├── environment/     # Environment, DB, server, toolkit base classes
├── evaluator/       # Task evaluation logic
├── gym/             # Gymnasium-compatible RL interface
├── hyper/           # The τ^τ outer loop (Client ⇄ Developer ⇄ sandbox construction wrapping the τ³ inner loop); registered as hyper_<base> domains for task discovery. The CLI entry point is `tau2 hyper-tau`.
├── knowledge/       # Knowledge retrieval pipeline (embedders, retrievers, postprocessors, sandbox)
├── metrics/         # Metrics computation
├── orchestrator/    # Simulation orchestrators (half-duplex, full-duplex)
├── registry.py      # Global registry for agents, domains, tasks, users
├── runner/          # Simulation runner (batch execution, checkpointing, build helpers)
├── scripts/         # CLI command implementations
├── user/            # User simulator implementations
├── utils/           # Shared utilities
└── voice/           # Voice synthesis, transcription, audio-native providers

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sierra-research/hyper-tau-bench](https://github.com/sierra-research/hyper-tau-bench) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
