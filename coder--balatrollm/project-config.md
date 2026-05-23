---
trigger: always_on
description: Manages Jinja2 strategy templates. Each strategy folder contains: `STRATEGY.md.jinja` (game rules/tactics), `GAMESTATE.md.jinja` (current state), `MEMORY.md.jinja` (action history), `TOOLS.json` (function calling tools), `manifest.json` (metadata). Current: `default` (conservative, financially-focused).
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

**GitHub Repository**: [`coder/balatrollm`](https://github.com/coder/balatrollm)

## Overview

BalatroLLM is an LLM-powered bot that autonomously plays Balatro using strategic decision-making. It uses the BalatroBot API (a Lua mod exposing JSON-RPC 2.0 endpoints) to control the game while leveraging large language models for gameplay decisions.

**Key capabilities:**

- Autonomous gameplay from start to completion
- LLM-driven strategic decision-making using OpenAI function calling
- Configurable strategies via Jinja2 templates
- Parallel execution of multiple game instances
- Comprehensive data collection for analysis and benchmarking

### Make Commands

Available make targets:

| Target           | Description                                             |
| ---------------- | ------------------------------------------------------- |
| `make help`      | Show all available targets                              |
| `make lint`      | Run ruff linter (check only)                            |
| `make format`    | Run ruff and mdformat formatters                        |
| `make typecheck` | Run type checker                                        |
| `make quality`   | Run all code quality checks (lint + typecheck + format) |
| `make all`       | Run all quality checks                                  |
| `make install`   | Install dependencies                                    |

**Important rules:**

1. **Only run make commands when explicitly asked.** Do not proactively run `make quality`, `make lint`, etc.
2. **Never run bare linting/formatting/typechecking tools.** Always use make targets instead:
    - Use `make lint` instead of `ruff check`
    - Use `make format` instead of `ruff format`
    - Use `make typecheck` instead of `ty check`
    - Use `make quality` for all checks combined

## Architecture

### Python Components (`src/balatrollm/`)

The bot is structured as a pipeline: CLI → Executor → Bot → (LLMClient + BalatroClient) → Game.

#### CLI (`cli.py`)

Entry point for the `balatrollm` command. Parses CLI arguments and YAML config files, generates task combinations from game parameters (seed, deck, stake, strategy, model). Config precedence: environment variables < YAML < CLI args.

#### Bot (`bot.py`)

Core game loop with LLM-powered decision-making. Handles game states (SELECTING_HAND, SHOP, ROUND_EVAL, BLIND_SELECT, GAME_OVER). Each decision: sends game state + strategy + memory to LLM → receives tool call → executes via BalatroClient. Tracks consecutive failures (max 3) and logs all data via Collector.

#### Client (`client.py`)

JSON-RPC 2.0 HTTP client for BalatroBot mod. Async client to `http://127.0.0.1:12346` (default 30s timeout). Methods: `gamestate`, `start`, `select`, `cash_out`, `play`, `buy`, `sell`, `use`, `screenshot`, etc.

#### LLMClient (`llm.py`)

Async OpenAI client wrapper with retry logic. Provider-agnostic (works with OpenAI, OpenRouter, any compatible API). Default: OpenRouter. Exponential backoff (max 3 retries), aborts after 3 consecutive timeouts. Default timeout: 240s.

#### StrategyManager (`strategy.py`)

Manages Jinja2 strategy templates. Each strategy folder contains: `STRATEGY.md.jinja` (game rules/tactics), `GAMESTATE.md.jinja` (current state), `MEMORY.md.jinja` (action history), `TOOLS.json` (function calling tools), `manifest.json` (metadata). Current: `default` (conservative, financially-focused).

#### Config (`config.py`)

Multi-source configuration. Precedence: environment variables < YAML < CLI args. Key env vars: `BALATROLLM_API_KEY`, `BALATROLLM_BASE_URL`, `BALATROLLM_MODEL`, `BALATROLLM_SEED/DECK/STAKE`, `BALATROLLM_STRATEGY`, `BALATROLLM_PARALLEL`, `BALATROLLM_HOST/PORT`.

#### Executor (`executor.py`)

Orchestrates parallel execution. Manages pool of Balatro instances via `balatrobot` package, assigns consecutive ports (12346, 12347, ...), uses task queue with worker pool pattern.

#### Collector (`collector.py`)

Data collection and statistics. Output: `runs/v1.0.0/{strategy}/{vendor}/{model}/{timestamp}_{deck}_{stake}_{seed}/`. JSONL format, OpenAI Batch API compatible.

### Strategy System

Template-based system using Jinja2. Templates render strategy guidance, game state, and action history into LLM prompts. At each decision point: render templates → send to LLM with tools → execute tool call. Custom strategies: create folder in `src/balatrollm/strategies/` with required template files.

### Relationship with BalatroBot

**BalatroBot** is a Lua mod that runs inside the Balatro game and exposes a JSON-RPC 2.0 HTTP API for programmatic control.

**How BalatroLLM uses BalatroBot:**

```
┌─────────────────────────────────────────┐
│          BalatroLLM (Python)            │
│                                         │
│  ┌────────┐      ┌──────────────────┐   │
│  │  Bot   │──────│   LLMClient      │   │◄──── OpenAI API
│  └───┬────┘      └──────────────────┘   │      (OpenRouter, etc.)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [coder/balatrollm](https://github.com/coder/balatrollm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
