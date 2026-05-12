---
trigger: always_on
description: - Dead-simple LLM benchmarking CLI. Point it at any OpenAI-compatible API and get TTFT (time to first token), throughput (tokens/second), p50/p95/p99 latencies, cost estimation, and quality scoring in seconds. Compare models side by side with winner highlights.
---

# AGENTS.md - bench-my-llm

## Overview
- Dead-simple LLM benchmarking CLI. Point it at any OpenAI-compatible API and get TTFT (time to first token), throughput (tokens/second), p50/p95/p99 latencies, cost estimation, and quality scoring in seconds. Compare models side by side with winner highlights.
- For developers choosing between LLM models/providers who need real performance numbers, not marketing claims.
- Core value: streaming TTFT measurement, built-in prompt suites (reasoning, coding, creative, factual), side-by-side model comparison with trophy indicators, Rich terminal reports, JSON export for CI pipelines.

## Architecture

```
+----------------+     +------------------+     +-------------------+
|   CLI (click)  | --> |   Runner         | --> | OpenAI-compatible |
|  run, compare, |     | run_benchmark()  |     |   API endpoint    |
|  models, suites|     | run_single_prompt|     | (OpenAI, Ollama,  |
|  report        |     | (streaming)      |     |  vLLM, Together)  |
+----------------+     +------------------+     +-------------------+
       |                       |
       v                       v
+----------------+     +------------------+
|   Reporter     |     |   Metrics        |
| print_report() |     | compute_metrics()|
| (Rich tables)  |     | estimate_cost()  |
+----------------+     | score_quality()  |
       |               | LatencyStats     |
       v               +------------------+
+----------------+
|   Compare      |
| compare_runs() |
| (side-by-side) |
+----------------+
```

**Data flow:**
1. CLI receives model name, suite, and options
2. `run_benchmark()` creates an OpenAI client, iterates through prompts
3. Each prompt is sent with `stream=True` to measure TTFT (time from request start to first chunk)
4. Chunks are collected, total latency and tokens/sec computed
5. `compute_metrics()` aggregates results: percentile latencies (numpy), cost estimation (model-specific pricing), quality scoring (Jaccard overlap with reference answers)
6. `print_report()` renders Rich tables. `compare_runs()` shows head-to-head with trophy icons.

## Directory Structure

```
bench-my-llm/
  .github/workflows/ci.yml        -- CI: lint + test on Python 3.10-3.12
  src/bench_my_llm/
    __init__.py                    -- __version__ = "0.1.1"
    __main__.py                    -- python -m bench_my_llm entry
    cli.py                         -- Click CLI: run, compare, models, suites, report
    runner.py                      -- BenchmarkRun, BenchmarkResult, run_benchmark(), run_single_prompt()
    metrics.py                     -- RunMetrics, LatencyStats, compute_metrics(), estimate_cost(), score_quality(), COST_TABLE
    prompts.py                     -- Prompt, PromptSuite, built-in suites (reasoning, coding, creative, factual, all)
    reporter.py                    -- print_report() with Rich panels and tables
    compare.py                     -- compare_runs() with trophy-annotated head-to-head table
  tests/                           -- 183 tests across 10 test files
    test_bench.py                  -- Core benchmark tests
    test_cli.py                    -- CLI command tests
    test_cli_extended.py           -- Extended CLI tests
    test_compare.py                -- Comparison tests
    test_reporter.py               -- Reporter output tests
    test_runner_mock.py            -- Runner tests with mocked API
    test_runner_extended.py        -- Extended runner tests
    test_metrics_extended.py       -- Metrics computation tests
    test_new_features.py           -- New feature tests
    test_error_paths.py            -- Error handling tests
  pyproject.toml                   -- Hatchling build, metadata
  README.md                        -- Full docs
  ROADMAP.md                       -- v0.2 plans
  CONTRIBUTING.md                  -- Contribution guidelines
  GETTING_STARTED.md               -- Quick start guide
  LICENSE                          -- MIT
```

## Core Concepts

- **BenchmarkResult**: Single prompt result. Fields: model, prompt_text, category, response_text, ttft_ms, total_latency_ms, tokens_generated, tokens_per_second, prompt_tokens, completion_tokens, reference.
- **BenchmarkRun**: Collection of results from a full run. Has model, suite_name, base_url, timestamp. Methods: `save(path)`, `load(path)`.
- **LatencyStats**: Percentile stats: p50_ms, p95_ms, p99_ms, mean_ms, min_ms, max_ms. Computed via `numpy.percentile()`.
- **RunMetrics**: Aggregated metrics: ttft (LatencyStats), total_latency (LatencyStats), mean_tps, median_tps, total_prompt_tokens, total_completion_tokens, estimated_cost_usd, mean_quality_score.
- **COST_TABLE**: Dict mapping model prefixes to (input_per_1k, output_per_1k) USD tuples. Covers OpenAI, Anthropic, Meta (Llama), Mistral, Google (Gemini), DeepSeek. Unknown models fall back to DEFAULT_COST = (0.002, 0.008).
- **score_quality()**: Jaccard word overlap between response and reference. Returns 1.0 if no reference provided.
- **Prompt**: Frozen dataclass with text, category, reference, max_tokens.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ManasVardhan/bench-my-llm](https://github.com/ManasVardhan/bench-my-llm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-09 -->
