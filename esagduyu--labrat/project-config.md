---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Test
uv run pytest                                    # full suite (~670 tests)
uv run pytest tests/unit/test_agent_loop.py      # single file
uv run pytest -k "test_smoke"                    # by name
uv run pytest --co -q                            # list tests without running

# Lint / format / types — run all three before committing
uv run ruff format .          # auto-fixes formatting (run this first)
uv run ruff check .           # linting (must be clean)
uv run pyright                # type checking (must be clean)

# Run the app
uv run labrat

# dbt-CI Scent pairing (docs/dbt-ci-pairing.md)
uv run labrat scent check     # read-only dbt<->Scent fingerprint staleness gate (offline dbt parse; exit 0/1)
uv run labrat scent ingest    # headless fix: re-ingest dbt semantics into project Scent
uv run labrat scent init-ci   # scaffold the GitHub Actions workflow

# Evals
uv run python scripts/eval_duckdb.py             # no API key needed
uv run scripts/eval_ade_bench.py --tasks helixops_saas001   # wrapper; needs ADE_BENCH_DIR + Docker
uv run python scripts/eval_dab.py --datasets stockindex,stockmarket   # DAB (needs ~/repos/DataAgentBench)
uv run python scripts/eval_dab.py --output-dir runs/dab/dab-<id>      # resume a crashed run
# DAB driver/provider selection + sandbox/scoring details: docs/dab-integration.md

# Standalone LabRat agent on any query (any provider):
uv run python scripts/run_task.py --prompt "..." \
    --connections '{"main":{"db_type":"duckdb","db_path":"/path.duckdb"}}' \
    --provider anthropic --model claude-sonnet-4-6

# Run the LabRat MCP server (mount inside any MCP-supporting host):
LABRAT_MCP_CONNECTIONS='{"main":{"db_type":"duckdb","db_path":"/path.duckdb"}}' \
    uv run python -m labrat.mcp.server
```

`asyncio_mode = "auto"` is set globally — no `@pytest.mark.asyncio` needed.
LLM-gated tests are skipped unless `ANTHROPIC_API_KEY` or `LABRAT_RUN_LLM_TESTS=1` is set.

## Architecture

### Agent loop (`src/labrat/agent/`)

`AgentLoop` in `loop.py` drives tool-use round-trips. It accepts a `ToolRegistry` and an LLM provider, sends messages, receives `TextBlock | ToolUseBlock` responses, dispatches tools, and feeds `ToolResultBlock`s back until the model stops calling tools. Optional `max_turns` and `max_tool_calls` cap the loop (both default `None` = unbounded). After `run()`, `loop.turns_used` and `loop.tool_calls_used` report what actually fired. `on_tool_call` (optional callback `(name, input, ok, output, latency_ms) → None`) fires per dispatch — the DAB `labrat-agent` path uses it to write per-call traces to `agent_tool_calls.jsonl`.

**Verifier loop (opt-in):** at the would-be-final turn (no tool calls), an optional `verifier` judges whether the answer addresses the question; if "insufficient" the feedback is injected as a new user turn and the loop continues — bounded by `max_verify_rounds` (default 2) AND the remaining turn budget. **Fail-open** (an unparseable verdict counts as sufficient, so it can never trap the loop). Types live in `verifier.py` (`Verdict`, `Verifier`, `LLMVerifier`, `parse_verdict`, `provider_llm_fn`), mirroring `validations.ValidationChecker`. `provider_llm_fn` adapts the loop's own `ModelProvider` (same model + billing). Exposed via `run_agent_task(verify=False)` — default off (costs an extra LLM call per would-be-final answer). Status goes to `on_status`, separate from `on_text` so it never corrupts `final_text`. (Measured no-benefit on DAB GPT-5.5 — see `docs/dab-progress-report.md` §Phase 6.)

> **This sufficiency-judge verifier is NOT the verification we're building next.** It judges *plausibility* ("does the answer address the question"), which measured no benefit. The **next build** (FEATURE_ROADMAP **T1a**, the #1 competitor-proven lever — both top-2 DAB teams verify, we don't) is a separate **verification layer**: K-of-N **consensus** + an independent **re-derive** stage, integrated **driver-agnostically at `DabSuite.run_trial`** (so it hits the `claude-mcp` leaderboard path, not just `run_agent_task`). Spec'd + planned on branch `feat/verification-layer` — see `docs/superpowers/{specs,plans}/2026-06-24-verification-layer*.md`. Memory: `project_verification_layer_next`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [esagduyu/labrat](https://github.com/esagduyu/labrat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-08 -->
