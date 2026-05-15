---
trigger: always_on
description: Guidance for Claude Code when working in this repository.
---

# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this project is

**Calibrate** (`calibrate-agent` on PyPI) is an open-source evaluation framework
for voice agents. It benchmarks LLMs, STT providers, TTS providers, and runs
agent simulations — all from a single CLI / Python library.

- Website / docs: https://calibrate.artpark.ai
- Built on top of [pipecat](https://github.com/pipecat-ai/pipecat).
- The CLI entry point is `calibrate` (defined in `pyproject.toml:scripts` →
  `calibrate.cli:main`).

The repo also ships an **Ink (React) terminal UI** in `ui/` that is bundled into
the Python package and launched from the CLI.

## Repository layout

```
calibrate/                 # Python package (the importable library + CLI)
├── cli.py                 # Top-level CLI entry — wires subcommands to UI/SDK
├── connections.py         # TextAgentConnection — HTTP client for external agents
├── judges.py              # text_judge / audio_judge / simulation_judge — LLM-as-judge core
├── langfuse.py            # Optional Langfuse tracing wrappers (@observe)
├── status.py              # Run-status reporting helpers
├── utils.py               # Provider language code maps, logging, validation
├── stt/
│   ├── eval.py            # Per-provider transcribe_* + transcribe_audio router
│   ├── metrics.py         # WER + LLM-judge aggregation (get_llm_judge_score)
│   ├── benchmark.py       # Multi-provider parallel runner + leaderboard
│   └── leaderboard.py     # Excel workbook generator
├── tts/
│   ├── eval.py            # Per-provider synthesize_* + synthesize_speech router
│   ├── metrics.py         # Audio LLM-judge aggregation (get_tts_llm_judge_score)
│   ├── benchmark.py       # Multi-provider parallel runner + leaderboard
│   └── leaderboard.py
├── llm/
│   ├── run_tests.py       # Tool-call / response evaluation across test cases
│   ├── run_simulation.py  # Multi-turn user-simulator conversations
│   ├── benchmark.py       # Multi-model parallel runner + leaderboard
│   ├── tests_leaderboard.py
│   ├── simulation_leaderboard.py
│   ├── metrics.py
│   └── _output.py         # Shared print_benchmark_summary
├── agent/
│   ├── bot.py             # Pipecat bot bootstrap
│   ├── run_simulation.py  # Voice-agent simulation driver
│   └── test.py            # Voice-agent tests
└── integrations/
    └── smallest/          # Smallest.ai STT/TTS provider integration

tests/                     # Test suite — mirrors the calibrate/ structure
├── stt/        test_eval.py, test_metrics.py, test_leaderboard.py
├── tts/        test_eval.py, test_metrics.py, test_leaderboard.py
├── llm/        test_benchmark.py, test_run_tests.py, test_run_simulation.py,
│               test_run_simulation_integration.py, test_output.py,
│               test_tests_leaderboard.py
├── test_connections.py, test_cli.py, test_judges.py,
│   test_sdk_judge_regressions.py

ui/                        # Ink (React + TypeScript) terminal UI
├── source/                # *.tsx entry points (app, llm-app, sim-app, etc.)
├── tests/                 # vitest tests
└── package.json           # Bundled into calibrate/ui/cli.bundle.mjs

docs/                      # Mintlify docs site (.mdx)
examples/                  # Example datasets + scripts users can run
.github/workflows/         # tests.yml, publish.yml, claude.yml, claude-code-review.yml
.githooks/pre-commit       # Runs pytest before commits to main
```

## Conventions in this codebase

### Evaluator dicts everywhere
Every LLM/audio judge in the codebase takes a list of **evaluator** dicts of
this shape:

```python
{
  "name": "semantic_match",
  "system_prompt": "...",
  "judge_model": "openai/gpt-4.1",   # routed through OpenRouter
  "type": "binary" | "rating",       # binary is the default if absent
  "scale_min": 1, "scale_max": 5,    # only for rating
}
```

Helpers in `calibrate/judges.py`:
- `is_rating(evaluator)` — True if `type == "rating"`
- `evaluator_result_value(ev, row)` — pulls the score/match value out of a per-row result
- `DEFAULT_STT_EVALUATOR`, `DEFAULT_TTS_EVALUATOR`, `DEFAULT_LLM_TEST_EVALUATOR`

Result shape returned by `text_judge`/`audio_judge`:
```python
{
  "evaluator_name": {"reasoning": str, "match": bool}   # binary
  "evaluator_name": {"reasoning": str, "score": int}    # rating
}
```

### Aggregation shape
`get_llm_judge_score` / `get_tts_llm_judge_score` return:
```python
{
  "scores": {
    "name": {"type": "binary", "mean": 0.85}                            # binary
    "name": {"type": "rating", "mean": 4.0, "scale_min": 1, "scale_max": 5}  # rating
  },
  "score": float,        # mean across evaluator means (legacy top-level)
  "per_row": [ ... ],    # list of per-row dicts, same shape as text_judge output
}
```

Leaderboards detect evaluators in `metrics.json` by looking for dict values
with a `type` field — that's the marker. `wer` and `ttfb` are top-level floats
and dicts respectively.

### Routing pattern
Both `transcribe_audio` (STT) and `synthesize_speech` (TTS) are dispatch
routers wrapped in `@backoff.on_exception(...)` + `@observe(...)`. They look up
the per-provider implementation in a dict and `await` it. For unit testing,
call `router.__wrapped__(...)` to skip the decorators (the `@backoff` retry

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ARTPARK-SAHAI-ORG/calibrate](https://github.com/ARTPARK-SAHAI-ORG/calibrate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
