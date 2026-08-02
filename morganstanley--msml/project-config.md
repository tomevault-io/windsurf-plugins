---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Alpha Lab is an autonomous research agent that explores datasets, generates analysis scripts, builds evaluation frameworks, and runs GPU-scale experiments — all without human intervention.

**Current state:** Full 4-phase pipeline (Phase 0 → 1 → 2 → 3) with local GPU executor. Supports OpenAI and Anthropic Claude (via Anthropic API). Domain-agnostic via adapter system — ships with built-in adapters for time series prediction, CUDA kernel optimization, and NanoGPT speed competition.

## Commands

```bash
# Run the full pipeline (headless) — OpenAI, default time_series domain
python run.py --config data/exchange_config.json --workspace ./workspace

# Run the full pipeline (headless) — Claude via Anthropic API
python run.py --config data/demo_exchange_config.json --workspace ./workspace_claude

# Run with a specific domain (copies built-in, then customizes via LLM)
# Set "domain": "cuda_kernel" or "domain": "nanogpt" in config JSON

# Run with a novel domain (triggers Phase 0 agent to generate adapter from scratch)
# Set "domain": "your task description here" in config JSON

# Run the web dashboard
python serve.py --config data/exchange_config.json --workspace ./workspace

# Generate synthetic test data
python data/generate_synthetic.py
```

Requires Python 3.11+ with dependencies from requirements.txt.

## Architecture

### Domain Adapter System (`adapter.py`, `adapter_loader.py`, `adapters/`)
Every domain-specific aspect of the pipeline is parameterized by a `DomainAdapter`:
- **Prompts**: 9 prompt `.md` files (one per phase/role), plus `domain_knowledge.md`
- **Metrics**: `MetricConfig` — primary_metric, direction (maximize/minimize), display_name
- **Experiment structure**: `ExperimentStructure` — required_files, entry_point, framework_dir

Three built-in adapters ship under `src/alpha_lab/adapters/`:
- **time_series** — Sharpe ratio (maximize), walk-forward backtesting
- **cuda_kernel** — throughput_gflops (maximize), benchmark framework
- **nanogpt** — wall_clock_seconds (minimize), training framework

Adapter resolution priority: workspace adapter > built-in matching domain > time_series fallback.

### Phase 0: Adapter Resolution & Customization (`phase0.py`)
Runs before Phase 1 to resolve, customize, or generate the domain adapter:
1. **Resume path**: `{workspace}/adapter/manifest.json` exists → load and return (already customized)
2. **Built-in match**: domain matches built-in name → copy template → run customization agent → return
3. **Default path**: no domain specified → copy time_series template → run customization agent → return
4. **Generation path**: free-text domain → run full generation agent with `write_adapter_file` + `read_reference_adapter` tools

The customization agent (`_run_customization_agent`) examines the actual data/task and patches adapter files to be task-specific. Uses `read_adapter`, `shell_exec`, `read_file`, and `patch_adapter_file`. Highest-value target is `domain_knowledge.md` (injected into every phase). Runs at `reasoning_effort="medium"`.

### Supervisory Agent (`supervisor.py`)
Meta-agent that monitors pipeline phases and can patch the adapter:
- `validate_adapter()` — after Phase 0 (always, for all domains): checks completeness, validity, prompt quality
- `review_phase1()` — after Phase 1: checks learnings, data report, scripts
- `review_phase2()` — after Phase 2: checks framework, tests, review verdict
- `phase3_health_check()` — during Phase 3: triggered when error rate > 40%, diagnoses systemic issues and patches adapter files (with git checkpoint)

### Provider System (`provider.py`, `provider_openai.py`, `provider_anthropic.py`)
All LLM calls go through the `Provider` protocol. Two implementations:
- **OpenAIProvider**: Wraps OpenAI Responses API. Built-in `web_search_preview`.
- **AnthropicProvider: Wraps Anthropic Messages API (Claude). Translates tool schemas from OpenAI format. Web search proxied through GPT.

The `get_provider()` factory in `client.py` returns the right provider based on config.

### Agent Loop (`agent.py`)
Provider-agnostic iterative loop: call `provider.stream_response()` → collect text + tool calls → dispatch tools → feed results back via `provider.build_tool_result_items()`. Conversation history tracked locally in `_input_history`. Accepts optional `adapter` param, passed to `execute_tool()`. Continues until `report_to_user` is called or `ask_user` returns control.

### Client (`client.py`)
Factory for providers:
- **OpenAI**: Standard OpenAI API with `OPENAI_API_KEY`
- **Anthropic: Anthropic Messages API with ANTHROPIC_API_KEY

### Local GPU Executor (`local_gpu.py`)
Replaces SLURM for single multi-GPU boxes. Same 5-method interface as `SlurmManager`:
- Spawns experiments as subprocesses with `CUDA_VISIBLE_DEVICES` pinning
- Polls `proc.poll()` for status
- Enforces time limits

### Tool System (`tools.py`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [morganstanley/MSML](https://github.com/morganstanley/MSML) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
