---
trigger: always_on
description: > This file provides context for Claude Code. It is loaded automatically when you run `claude` from a project that uses `agent-eval`.
---

# agent-eval — AI Assistant Context

> This file provides context for Claude Code. It is loaded automatically when you run `claude` from a project that uses `agent-eval`.

---

## What is agent-eval?

`agent-eval` is an evaluation CLI for ADK agents. It acts as the glue between OpenTelemetry traces from ADK, Vertex AI Evaluation for LLM-as-judge metrics, and Gemini for analysis.

**The evaluation cycle:**
1. **Interact** — Generate agent traces. Either `agent-eval simulate` (UserSim, multi-turn) or `agent-eval interact` (DIY, single-turn) — both are forms of interacting with the agent, just different drivers.
2. **Evaluate** — Score traces with deterministic metrics (latency, tokens, cost) + LLM-as-judge metrics (quality, accuracy)
3. **Analyze** — Generate AI-powered root cause analysis

**Context Engineering Principles** guide optimizations:
- **Offload**: Move deterministic logic out of the LLM into tools/code
- **Reduce**: Summarize or compact stale context to prevent "context rot"
- **Retrieve**: Replace static knowledge with RAG
- **Isolate**: Split monolithic agents into specialized sub-agents
- **Cache**: Restructure prompts for prefix caching

---

## Typical Project Structure

`agent-eval init` writes ONE unified `tests/eval/` folder at the **agent project root** (the directory with `pyproject.toml`) — NEVER inside the agent module. There is one source of truth — `tests/eval/dataset.jsonl` — that feeds every command:

- `simulate` reads multi-turn rows (rows with `history` or `conversation_plan`)
- `interact` reads single-turn rows
- `agent-engine` reads single-turn rows (skips multi-turn with a clear pointer at `simulate`)
- `evaluate` consumes whichever interaction file the previous step wrote

```
my-agent/
├── my_agent/                                  # ADK agent source code
│   ├── agent.py
│   ├── tools/
│   └── ...
├── tests/eval/                                # Unified — ONE source of truth
│   ├── dataset.jsonl                          # All rows (prompt / response / reference / history / intermediate_events / session_inputs / expected_*)
│   ├── metrics/metric_definitions.json        # LLM-as-judge metric rubrics
│   └── results/                               # Evaluation outputs (eval_summary.json, gemini_analysis.md, ...)
├── pyproject.toml
└── .env
```

ADK's `adk eval` runtime expects `conversation_scenarios.json`, `session_input.json`, and `eval_config.json` next to `agent.py`. `agent-eval simulate` projects those from `dataset.jsonl` at run time and writes them inside `my_agent/` as **ephemeral cache** (gitignore them). Users never edit those files — they edit `dataset.jsonl`.

> **Migrating an old project?** Run `agent-eval migrate` to fold every legacy location into the unified file:
> - `<agent>/eval/scenarios/conversation_scenarios.json` (legacy UserSim)
> - `<agent>/eval/eval_data/golden_dataset.json` (legacy DIY)
> - `<agent>/eval/metrics/metric_definitions.json` (relocated to `<project_root>/tests/eval/metrics/`)
> - `<agent>/tests/eval/dataset.jsonl` (the wrongly-placed F3 location from pre-rescue scaffolds — folded into the canonical project-root location, source removed)
> Originals are copied to `<project_root>/tests/eval/.backup/<timestamp>/` first.

---

## CLI Commands

| Command | Purpose |
|---------|---------|
| `uv run agent-eval setup` | Walk gcloud auth, ADC, project + location, Vertex AI API enablement, autorater IAM (run once per shell) |
| `uv run agent-eval init` | Scaffold eval files (with optional AI metric generation via `--ai-metrics`) |
| `uv run agent-eval migrate` | Fold legacy `eval/scenarios/` + `golden_dataset.json` AND any wrongly-placed `<agent>/tests/eval/dataset.jsonl` (F3) into the canonical `<project_root>/tests/eval/dataset.jsonl`. Idempotent. |
| `uv run agent-eval import --from <evalset>.json` | Flatten an ADK `.evalset.json` file into `tests/eval/dataset.jsonl` |
| `uv run agent-eval run` | Full pipeline: simulate + interact + evaluate + analyze |
| `uv run agent-eval simulate` | Run ADK User Sim + convert traces (multi-turn) |
| `uv run agent-eval interact` | Run queries against a live agent endpoint (single-turn) |
| `uv run agent-eval evaluate` | Run deterministic + LLM-as-judge metrics (supports multiple `--interaction-file`) |
| `uv run agent-eval agent-engine` | Streamlined `create_evaluation_run()` against a deployed Agent Engine (single-turn, additive when deployed) |
| `uv run agent-eval analyze` | Generate AI-powered analysis reports |
| `uv run agent-eval convert` | Convert ADK traces to evaluation format (used by simulate) |
| `uv run agent-eval create-dataset` | Convert ADK test files to golden dataset format |
| `uv run agent-eval dashboard` | Launch interactive Gradio dashboard for comparing runs |

---

## Common Tasks You Should Help With

### Interpreting Evaluation Results

- `eval_summary.json` → Aggregated metrics (start here)
- `gemini_analysis.md` → AI root cause analysis
- `question_answer_log.md` → Per-question detailed breakdown with scores

### Debugging Evaluation Issues


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GoogleCloudPlatform/professional-services](https://github.com/GoogleCloudPlatform/professional-services) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
