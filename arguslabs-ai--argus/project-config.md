---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## NOTE: ALWAYS REMEMBER
Never add a "co-authored by claude" while pushing or in CI/CD operation. It should always look like no AI assisted coding was used.

## Commands

```bash
# Install in editable mode with dev dependencies
pip install -e ".[dev]"

# Run tests
pytest tests/

# Run single test
pytest tests/test_smoke.py::test_name -v

# Run tests with coverage
pytest --cov=src --cov-report=term-missing

# Lint
ruff check src/

# Format
ruff format src/

# Type check
mypy src/argus

# Run the CLI
argus --help
argus show <run-id>
argus check last
argus diff <run-a> <run-b>
argus replay <run-id> <node>
argus ui
pytest --argus                       # fail tests whose ARGUS run was not clean
```

## Architecture

ARGUS is a production readiness platform for AI agent pipelines — detects silent failures, semantic degradation, and contract violations before deployment (LangGraph-first, framework-agnostic).

### Detection Pipeline

Every wrapped node executes through this pipeline:
1. Output captured and serialized
2. **Tool failure scan** (`inspector.py`): error keys, HTTP status codes, empty results, semantic registry
3. **Structural inspection** (`inspector.py`): missing required fields vs successor type hints, type mismatches
4. **Semantic validators**: custom per-node or wildcard validators
5. **Anomaly detection**: behavioral anomaly signals (output size, timing, structure)
6. **LLM semantic judge** (`semantic_checker.py`): evidence-aware final ruling — receives all prior signals (validator results, anomaly signals, inspection findings) as context. Cannot override validator failures or critical anomalies. Returns `evidence_considered` and `overridden_signals` for audit trail.
7. Status assigned: `pass | fail | crashed | semantic_fail | interrupted`
8. `NodeEvent` recorded; auto-finalize if last node or error

### Core Classes

- **`ArgusSession`** (`session.py`): Framework-agnostic core. Thread-safe, supports sync + async. Wrap nodes via `wrap()`, `instrument()`, or `@session.node()` decorator.
- **`ArgusWatcher`** (`watcher.py`): LangGraph adapter. Accepts a `StateGraph` as optional first arg: `ArgusWatcher(graph)` auto-attaches, or use `watch()`/`watch_compiled()` separately. All config params are keyword-only. Supports `record_http=True` for deterministic reruns.
- **`ArgusInspector`** (`inspector.py`): Static functions for `inspect_tool_outputs()`, `inspect_transition()`, and `build_root_cause_chain()`. Root cause analysis walks backward through events to find where a failing field was omitted.

### Key Files

| File | Role |
|------|------|
| `src/argus/session.py` | Core monitoring session, wraps arbitrary callables |
| `src/argus/watcher.py` | LangGraph adapter (thin wrapper over `ArgusSession`) |
| `src/argus/pytest_instrument.py` | pytest `--argus` auto-wrap of `StateGraph.compile()` / compiled `invoke()` |
| `src/argus/inspector.py` | Silent failure detection + root cause chain |
| `src/argus/registry.py` | Semantic signature registry for LLM output heuristics |
| `src/argus/models.py` | Dataclasses: `NodeEvent`, `RunRecord`, `InspectionResult`, `LLMUsage` |
| `src/argus/storage.py` | Persist/load `RunRecord` to `.argus/runs/<run-id>.json` |
| `src/argus/patcher.py` | Patch LangGraph node functions; handle LG 0.2+ and legacy formats |
| `src/argus/llm_tracker.py` | Extract token usage from node output metadata |
| `cloud/pricing.py` | Cost calculation per model (enterprise) |
| `src/argus/http_recorder.py` | HTTP recording/playback for deterministic reruns |
| `src/argus/replay.py` | Rerun engine with reducer-aware state merging |
| `src/argus/llm_proxy.py` | Shared LLM transport — all chat completion calls go through here. Resolves BYOK (OpenAI/Anthropic/Google) first, falls back to hosted Supabase proxy |
| `src/argus/providers.py` | Per-provider request/response translation for BYOK (message format, model remapping, response normalization) |
| `src/argus/signature_generalizer.py` | Generalizes failure signatures via LLM + heuristic fallback. Uses `llm_proxy` for the LLM path |
| `src/argus/check.py` | CI gate: evaluate a `RunRecord` as clean vs crash / silent_failure / semantic_fail |
| `src/argus/cli/cmd_check.py` | `argus check last` / `argus check <id>` — exit 1 when the run was not clean |
| `src/argus/pytest_plugin.py` | pytest `--argus` plugin: fail tests whose instrumented invoke was not clean |
| `src/argus/cli/main.py` | `argus` CLI entry point (Typer) |
| `src/argus/cli/cmd_doctor.py` | `argus doctor` diagnostic command |
| `src/argus/data/signatures.json` | Bundled semantic failure signatures |

### Semantic Signature Registry (`registry.py` + `data/signatures.json`)

Detects placeholder/degraded LLM outputs using match strategies:
- `exact_ci`, `contains_ci`, `prefix_ci`: string matching
- `regex`: compiled pattern
- `repetition`: n-gram repetition detection

Categories: `placeholder_outputs`, `null_like_semantic`, `suspicious_phrases`, `corrupted_markers`, `repeated_filler_text`, `malformed_embedded_json` → mapped to `placeholder_detected` or `semantic_degradation` failure types.

### LLM Transport


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ArgusLabs-ai/ARGUS](https://github.com/ArgusLabs-ai/ARGUS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-21 -->
