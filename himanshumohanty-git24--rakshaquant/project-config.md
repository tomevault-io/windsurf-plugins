---
trigger: always_on
description: validates signals, and applies deterministic risk rules. The default mode is 100% free:
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

RakshaQuant is an agentic paper-trading system for the Indian NSE market. A LangGraph
pipeline of LLM-backed agents (via Groq) classifies the market regime, picks strategies,
validates signals, and applies deterministic risk rules. The default mode is 100% free:
YFinance market data + a local virtual wallet + the Groq free tier. DhanHQ (broker) and
PostgreSQL (memory) are optional. It is educational/paper-trading only.

## Commands

Dependency management is via [`uv`](https://github.com/astral-sh/uv). The distribution name is `trading-agent`; the import
package is `src` (all internal imports are `from src.<module> import ...`).

```bash
uv sync                         # install runtime deps
uv sync --extra dev             # install dev deps (pytest, ruff, mypy)

uv run python scripts/check_config.py       # validate .env / settings (run this first)
uv run python scripts/run_live_trading.py   # MAIN entry point: live/sim dashboard + paper execution
uv run python src/backtesting/engine.py     # run a backtest

# pytest/ruff/mypy live in the `dev` optional group — pass --extra dev (or `uv sync --extra dev` once).
uv run --extra dev pytest                     # full test suite (210 tests)
uv run --extra dev pytest tests/test_agents.py   # one file
uv run --extra dev pytest tests/test_agents.py::TestRiskCompliance::test_risk_compliance_no_signals   # one test
uv run --extra dev pytest --cov=src           # with coverage

uv run --extra dev ruff check .   # lint (line-length 100, rules E,F,I,N,W,UP)
uv run --extra dev ruff format .  # format
uv run --extra dev mypy src       # type-check (strict mode is enabled)
```

`pytest` is configured with `asyncio_mode = "auto"`, so `async def test_*` functions run
without an explicit `@pytest.mark.asyncio` decorator.

## Architecture

### Agent pipeline (the core)

The system is a [LangGraph](https://github.com/langchain-ai/langgraph) `StateGraph` built in [src/agents/graph.py](src/agents/graph.py).
A single `TradingState` (a `TypedDict` defined in [src/agents/state.py](src/agents/state.py))
flows through every node; each node returns a **partial dict** that LangGraph merges into state.
The pipeline:

1. **support_agents** — runs news, sentiment, and prediction agents to enrich state. All
   failures here are non-fatal (caught and logged); they only add context.
2. **market_regime** — LLM classifies regime (`trending_up/down`, `ranging`, `volatile`).
   Conditional edge: if `regime_confidence < 0.3` **or** the kill switch fires, the graph ends.
3. **strategy_selection** — picks active strategies for the regime.
4. **signal_validation** — filters raw signals. Conditional edge: if no signals survive, the graph ends.
5. **risk_compliance** — a **deterministic rules engine** (not an LLM) that does final approval,
   position sizing, and enforces limits. Populates `approved_trades` / `risk_rejected`.

To add an agent: write a `*_node(state) -> dict` function, register it with
`workflow.add_node(...)`, and wire edges in `create_trading_graph()`. Conditional routing
lives in `should_continue_after_*` predicate functions.

### LLM agent conventions

Every LLM node (see [src/agents/market_regime.py](src/agents/market_regime.py) as the
reference implementation) follows the same resilience pattern — **preserve it when editing or adding agents**:

- Acquire the shared rate limiter (`get_groq_limiter`) and circuit breaker
  (`get_groq_circuit_breaker`) before calling the LLM.
- Try `settings.groq_model_primary`, then fall back to `groq_model_fallback` on rate-limit (429) errors.
- On **any** failure (circuit open, rate limit, parse error), return a deterministic
  `_fallback_*` result instead of raising. The graph must never crash on a bad LLM call.
- LLM output is JSON; parsing strips ```` ```json ```` / ```` ``` ```` fences and clamps/validates fields.

**Support-agent state contracts.** The support agents enrich `TradingState` with keys the
regime/validation agents read; the *types must match* or the enrichment is silently dropped
(the consumer raises `TypeError`, which the agent's broad `except` swallows → it falls back
without the context). The canonical contracts (declared in [state.py](src/agents/state.py)):
`news_sentiment` is a **dict** `{"avg_sentiment": float}` (not a bare float), `market_mood` is
the full `SentimentSignal.to_dict()` dict (read `market_mood["mood_index"]`, not `market_mood`
itself), `news_headlines` is a list of `{"title","sentiment"}`, and `prediction_signals` is a
list of `PredictionSignal.to_dict()`. The prediction node sources from raw `signals` (populated
when support agents run), **not** `validated_signals` (still empty at that stage). When adding a
consumer, read with `isinstance`/`.get(...)` guards so a stray type never crashes the node.

### Configuration

All config is centralized in [src/config/settings.py](src/config/settings.py): a
pydantic-settings `Settings` model loaded from `.env`. Access it **only** through the cached
`get_settings()`; use `reload_settings()` to clear the cache. Secrets are `SecretStr` —

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [HimanshuMohanty-Git24/RakshaQuant](https://github.com/HimanshuMohanty-Git24/RakshaQuant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->
