---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

HolyEval is an open-source LLM evaluation framework. Reproduce any published benchmark with one command; extend with custom evaluators via a pluggable agent architecture.

## Commands

```bash
# Install dependencies (uv workspace)
uv sync

# Run benchmarks
python -m benchmark.basic_runner healthbench sample --target-model gpt-4.1
python -m benchmark.basic_runner healthbench full --target-model gpt-4.1 --limit 50
python -m benchmark.basic_runner healthbench hard --target-model gemini-3-pro -p 5
python -m benchmark.basic_runner medcalc sample --target-model gpt-4.1
python -m benchmark.basic_runner agentclinic medqa --target-model gpt-4.1
python -m benchmark.basic_runner memoryarena sample --target-model gpt-4.1
python -m benchmark.basic_runner eslbench sample50-20260324 --target-model gpt-4.1        # ESLBench quick (50 cases)
python -m benchmark.basic_runner eslbench full-20260324 --target-model gpt-4.1 -p 5       # ESLBench full (1800 cases)
python -m benchmark.basic_runner healthbench sample --target-model gpt-4.1 --ids hb_abc
python -m benchmark.basic_runner healthbench sample --target-model gpt-4.1 --limit 10 -p 3 -v
python -m benchmark.basic_runner healthbench sample --resume

# Data preparation (required before running ESLBench via CLI; automatic via Web UI)
python -m generator.eslbench.prepare_data            # download HF data + build per-user DuckDB
python -m generator.eslbench.prepare_data --force    # force re-download + rebuild

# Data conversion (external datasets → HolyEval format)
python -m generator.healthbench.converter input.jsonl output.jsonl --target-model gpt-4.1
python -m generator.medcalc.converter
python -m generator.agentclinic.converter <input.jsonl> <output.jsonl>
python -m generator.medhall.data_gen --count 15 --output generator/medhall/raw_data.jsonl
python -m generator.medhall.converter generator/medhall/raw_data.jsonl benchmark/data/medhall/theta.jsonl
python -m generator.memoryarena.converter

# Web UI
python -m web    # uvicorn :8000, auto-reload

# Tests
pytest evaluator/tests/
pytest evaluator/tests/test_e2e.py

# Lint
ruff check .
ruff format .
```

## Architecture

### Execution Flow

```
TestCase (JSON) → Orchestrator (do_single_test)
  1. Initialize agents from TestCase config via plugin registry
  2. Dialogue loop: TestAgent ↔ TargetAgent (until is_finished or max_turns)
  3. EvalAgent.run(memory_list, session_info) → EvalResult
  4. Return TestResult (score, pass/fail, feedback, cost)
```

All call paths (CLI, batch, API) funnel through `evaluator/core/orchestrator.py:do_single_test()`.

### Batch Execution

```python
session = BatchSession(cases, max_concurrency=5, on_progress=callback)
report = await session.run()       # Returns TestReport
session.snapshot()                  # JSON-serializable progress snapshot
session.cancel()                    # Cooperative cancellation
```

### Plugin System

Three agent types use `__init_subclass__` auto-registration:

```python
class CustomTestAgent(AbstractTestAgent, name="custom"):
    ...
# Lookup: AbstractTestAgent.get("custom")
```

Plugins activate on import (in `evaluator/plugin/`). The `core/` layer depends only on abstract interfaces.

| Agent Type | Interface | Built-in Plugins |
|---|---|---|
| **TestAgent** (virtual user) | `core/interfaces/abstract_test_agent.py` | `auto` (LLM-driven), `manual` (scripted) |
| **TargetAgent** (system under test) | `core/interfaces/abstract_target_agent.py` | `llm_api` (OpenAI/Gemini) |
| **EvalAgent** (evaluator) | `core/interfaces/abstract_eval_agent.py` | `semantic`, `healthbench`, `medcalc`, `hallucination`, `kg_qa`, `memoryarena` |

Add custom plugins by inheriting from the abstract base classes. Use `/add-eval-agent` or `/add-target-agent` skills for guided scaffolding.

#### Plugin Metadata

Plugins can declare class attributes for inspector discovery:

| Attribute | Applies to | Description |
|---|---|---|
| `_display_meta` | All | Display metadata: `icon`, `color`, `features` |
| `_cost_meta` | EvalAgent | `{"est_cost_per_case": float}` (USD/case) |
| `_cost_meta` | TargetAgent | `{"est_input_tokens": int, "est_output_tokens": int}` |
| `_config_model` | TestAgent | Config model class name in schema.py |

### Key Modules

- **`evaluator/core/schema.py`** — Pydantic v2 data models: TestCase, UserInfo, TargetInfo, EvalInfo, TestResult, SessionInfo
- **`evaluator/core/orchestrator.py`** — `do_single_test()`, `do_batch_test()`, `BatchSession`, `CaseContext`, `CaseStatus`
- **`evaluator/utils/llm.py`** — Unified LLM interface `do_execute()` via langchain. Supports OpenAI and Google Gemini
- **`evaluator/core/bench_schema.py`** — Benchmark models: BenchItem, BenchMark, BenchReport, conversion functions
- **`evaluator/utils/benchmark_reader.py`** — Read/load `benchmark/data/` (shared by CLI + Web)
- **`evaluator/utils/report_reader.py`** — Read/write `benchmark/report/` (shared by CLI + Web)
- **`evaluator/utils/agent_inspector.py`** — Reflect plugin registry for metadata (shared by CLI + Web)
- **`evaluator/utils/checkpoint.py`** — Checkpoint manager for resume-on-interrupt


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [healthmemoryarena/holyeval](https://github.com/healthmemoryarena/holyeval) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
