---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

LLM coding benchmark harness that runs autonomous coding sessions against a fixed Rails application brief. Compares local models (Ollama or llama-swap) and cloud models (via OpenRouter) under the same prompt, using `opencode run --agent build --format json` as the runner. Uses a two-phase flow: phase 1 builds the Rails app, phase 2 validates boot/Docker/Compose.

## Key Commands

```bash
# Run benchmark (default set: models not marked skip_by_default)
python scripts/run_benchmark.py

# Run specific model(s)
python scripts/run_benchmark.py --model claude_opus_4_6 --model kimi_k2_5

# Force re-run even if result.json exists
python scripts/run_benchmark.py --model gemma4_31b --force

# Rebuild report from existing results without running models
python scripts/run_benchmark.py --report-only

# Refresh local opencode benchmark config without running
python scripts/run_benchmark.py --sync-ollama-contexts-only

# Use llama-swap instead of Ollama for local models
python scripts/run_benchmark.py --local-backend llama-swap --local-api-base http://192.168.0.90:8080

# Warmup local Ollama models (probes context sizes)
python scripts/warmup_ollama_models.py
python scripts/warmup_ollama_models.py --api-base http://192.168.0.90:11434

# Runtime validation of generated projects (local boot, Docker, browser)
python scripts/analyze_results_runtime.py
```

## Architecture

### Package layout (`scripts/benchmark/`)

The benchmark logic lives in a Python package under `scripts/benchmark/`:

- `util.py` — shared helpers: JSON I/O, timestamps, formatting, HTTP requests
- `backends.py` — `LocalModelBackend` ABC with `OllamaBackend` and `LlamaSwapBackend` implementations. Handles preflight (unload, preload, health check) for local model servers.
- `config.py` — `BenchmarkConfig` dataclass, opencode config generation, project summarization, model selection helpers
- `runner.py` — `StreamResult` dataclass, process management (`stream_process_output`), phase execution (`run_opencode_phase`, `run_model`)
- `report.py` — report generation (`build_report`, `load_results`)

### Entry points

- `scripts/run_benchmark.py` — thin CLI that parses args, creates `BenchmarkConfig`, and delegates to the package
- `scripts/warmup_ollama_models.py` — probes Ollama models at candidate context sizes
- `scripts/analyze_results_runtime.py` — post-run validator (local boot, Docker build, Docker Compose, headless browser)
- `scripts/browser_probe.mjs` — Chromium CDP helper for runtime validation

### Config layer

- `config/models.json` — model registry with slugs, provider IDs, per-model overrides (`skip_by_default`, `benchmark_context_override`, `enable_followup`), and runner command definition
- `config/opencode.benchmark.json` — auto-generated local opencode config for benchmark isolation (never edit manually)
- `config/warmup_known.json` — seed data for warmup results (models already probed manually)

### Prompt layer

- `prompts/benchmark_prompt.txt` — phase 1 implementation prompt
- `prompts/benchmark_followup_prompt.txt` — phase 2 validation prompt

### Output per model (`results/<slug>/`)

- `project/` — generated workspace
- `result.json` — normalized metadata (status, elapsed, tokens, phases)
- `opencode-output.ndjson` / `opencode-stderr.log` — raw phase 1 output
- `followup-*` — phase 2 continuation output
- `session-export.json` — opencode session snapshot (when available)

### Reports

Auto-generated (rebuilt every benchmark run):
- `docs/report.md` — AMD server / cloud profile consolidated table
- `docs/report.nvidia.md` — NVIDIA RTX 5090 workstation profile consolidated table
- `docs/ollama_warmup.md` — Ollama warmup preflight tok/s
- `docs/llama_swap_warmup.nvidia.md` — NVIDIA llama-swap preflight tok/s

Hand-written deep code review (the actual interpretive analysis):
- `docs/success_report.md` — AMD/cloud profile per-model code audit, Tier 1/2/3 runtime viability, failure analysis (including Gemma 4 Ollama Cloud 504 timeout investigation), pricing/time/test comparison tables
- `docs/success_report.nvidia.md` — NVIDIA workstation profile audit + headline finding that Claude reasoning distillation does NOT transfer library API knowledge
- `docs/success_report.multi_model.md` — 7 multi-agent variants (3 Claude Code, 2 opencode, 2 Codex). Headline findings: zero delegations happened across all 7 runs; Claude Code's harness context made Opus 4.7 hallucinate `chat.complete` (Tier 3) vs opencode's Tier 1 on identical model+prompt

Local infra docs:
- `docs/llama-swap.md` — full guide to the NVIDIA llama-swap Docker setup (CUDA 12.8 + sm_120 build, model sourcing via Ollama symlinks vs HF GGUFs, VRAM budget reasoning, common pitfalls)
- `docs/codex-integration.md` — Codex CLI integration for GPT 5.4 (hurdles: shell wrapper needs `bash -lc`, relative paths need `.resolve()`, sandbox flags, reasoning effort via `-c`, different JSONL event format)

## Model Slug Convention


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [akitaonrails/llm-coding-benchmark](https://github.com/akitaonrails/llm-coding-benchmark) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
