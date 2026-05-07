---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

**GitHub Repository**: [`coder/balatrobench`](https://github.com/coder/balatrobench)

## Overview

BalatroBench is a benchmark analysis tool for BalatroLLM runs. It processes run data from `balatrollm` and generates structured benchmark results for leaderboards comparing models and strategies.

**Key capabilities:**

- Analyze BalatroLLM runs and compute aggregated statistics
- Generate model leaderboards (comparing models using same strategy)
- Generate strategy leaderboards (comparing strategies for same model)
- Extract per-request data (prompts, reasoning, tool calls, metadata)
- Optional PNG to WebP screenshot conversion

### Make Commands

Available make targets:

| Target           | Description                                             |
| ---------------- | ------------------------------------------------------- |
| `make help`      | Show all available targets                              |
| `make lint`      | Run ruff linter (check only)                            |
| `make format`    | Run ruff and mdformat formatters                        |
| `make typecheck` | Run type checker                                        |
| `make quality`   | Run all code quality checks (lint + typecheck + format) |
| `make install`   | Install dependencies                                    |

**Important rules:**

1. **Only run make commands when explicitly asked.** Do not proactively run `make quality`, `make lint`, etc.
2. **Never run bare linting/formatting/typechecking tools.** Always use make targets instead:
    - Use `make lint` instead of `ruff check`
    - Use `make format` instead of `ruff format`
    - Use `make typecheck` instead of `ty check`
    - Use `make quality` for all checks combined

## Architecture

### Python Components (`src/balatrobench/`)

The package is structured as a pipeline: CLI → Analyzer → Writer.

#### CLI (`cli.py`)

Entry point for the `balatrobench` command. Parses input/output directories, infers version from path. Orchestrates analysis for both models and strategies views, generates manifests.

```bash
# Analyze runs from a specific directory
balatrobench --input-dir /path/to/runs/v1.0.0

# Custom output directory
balatrobench --input-dir /path/to/runs/v1.0.0 --output-dir /path/to/output

# Enable WebP conversion
balatrobench --input-dir /path/to/runs/v1.0.0 --webp
```

#### Analyzer (`analyzer.py`)

Core analysis engine. Reads BalatroLLM run directories, aggregates statistics across runs, creates leaderboard entries.

**Two analysis modes:**

- `analyze_models()`: Group runs by strategy, compare models
- `analyze_strategies()`: Group runs by model, compare strategies

Computes: run counts, win rates, round averages/std, token usage, timing, costs.

#### Writer (`writer.py`)

File I/O for benchmark output. Writes JSON files for manifests, leaderboards, runs, and per-request data. Handles PNG→WebP conversion using `cwebp`.

#### Extractor (`extractor.py`)

Parses BalatroLLM JSONL files (`requests.jsonl`, `responses.jsonl`). Extracts:

- Request content: strategy, gamestate, memory prompts
- Response data: reasoning, tool calls
- Metadata: provider, tokens, timing, cost

#### Models (`models.py`)

Output dataclasses for benchmark files. Defines the schema for all generated JSON.

#### Source (`source.py`)

TypedDicts for reading BalatroLLM source files (`task.json`, `strategy.json`, `stats.json`).

#### Enums (`enums.py`)

Balatro game enums: `Deck` (15 types) and `Stake` (8 difficulty levels).

### Relationship with BalatroLLM

**BalatroLLM** runs games and writes data to `runs/`. **BalatroBench** reads that data and generates benchmark output.

```
┌─────────────────────────────────────────────────────────────────┐
│                    BalatroLLM (Input)                           │
│  runs/v1.0.0/{strategy}/{vendor}/{model}/{run_id}/              │
│    ├── task.json, strategy.json, stats.json                     │
│    ├── requests.jsonl, responses.jsonl                          │
│    └── screenshots/                                             │
└──────────────────────────┬──────────────────────────────────────┘
                           │ balatrobench
                           ▼
┌─────────────────────────────────────────────────────────────────┐
│                   BalatroBench (Output)                         │
│  site/benchmarks/                                               │
│    ├── models/{version}/{strategy}/leaderboard.json             │
│    └── strategies/{version}/{vendor}/{model}/leaderboard.json   │
└─────────────────────────────────────────────────────────────────┘
```

## Output Structure

```
site/benchmarks/
├── models/                                    # Compare MODELS (same strategy)
│   ├── manifest.json
│   └── {version}/{strategy}/
│       ├── leaderboard.json                   # ModelsLeaderboard
│       └── {vendor}/{model}.json              # Runs
│           └── {run_id}/{request_id}/
│               ├── metadata.json              # Request metadata
│               ├── tool_call.json             # LLM tool calls
│               ├── reasoning.md               # LLM reasoning

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [coder/balatrobench](https://github.com/coder/balatrobench) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
