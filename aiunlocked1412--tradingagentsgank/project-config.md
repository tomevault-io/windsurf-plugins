---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Common commands

### Install / run

**Preferred — `./run.sh` launcher** (handles venv bootstrap, port collisions, and HTTP health-check automatically):
```bash
./run.sh              # start web UI at 127.0.0.1:8787 (default)
./run.sh web          # explicit
./run.sh cli          # interactive Typer/Rich CLI
./run.sh script       # one-shot main.py
./run.sh stop         # stop the running web server
./run.sh logs         # tail web log
./run.sh status       # check if web is running
HOST=0.0.0.0 PORT=9000 ./run.sh web   # override host/port
```

`run.sh` writes the PID to `.run/web.pid` and logs to `.run/web.log`. It auto-creates `.venv` (prefers `uv`, falls back to `python3 -m venv`) and `pip install -e .` if missing — so a fresh clone runs with a single command.

**Manual / raw entry points:**
```bash
pip install .                        # install the package + console scripts
.venv/bin/python main.py             # one-shot scripted run (NVDA via OpenRouter, see file)
.venv/bin/python -m cli.main         # interactive CLI from source
.venv/bin/python -m tradingagents.webui.server --host 127.0.0.1 --port 8787
tradingagents                        # console script (only after `pip install`)
tradingagents-web                    # console script (only after `pip install`)
docker compose run --rm tradingagents
```

Note: the project uses a `.venv` at the repo root (Python 3.11 via `uv`). The `tradingagents` / `tradingagents-web` console scripts only exist **after** `pip install` — when working from source without install, use the `python -m ...` forms above (or just `./run.sh`).

CLI flags worth knowing (see `cli/main.py`):
- `tradingagents analyze --checkpoint` — opt into LangGraph SQLite checkpoint resume.
- `tradingagents analyze --clear-checkpoints` — wipe checkpoints before running.

### Tests
```bash
pytest                                          # full suite; markers configured in pyproject.toml
pytest -m unit                                  # fast unit-only
pytest -m smoke                                 # quick sanity tests
pytest -m "not integration"                     # skip tests that hit real services
pytest tests/test_memory_log.py                 # single file
pytest tests/test_memory_log.py::TestParseEntry # single class
pytest tests/test_memory_log.py -k "rotation"   # filter by name substring
```

`tests/conftest.py` autouses a fixture that injects placeholder API keys for every supported provider, so the suite runs without credentials. There is no separate lint config — `pyproject.toml` only configures `pytest`.

### Diagnostics
```bash
# Verify a provider's structured-output path end-to-end (Research Manager → Trader → Portfolio Manager → SignalProcessor)
OPENAI_API_KEY=... python scripts/smoke_structured_output.py openai
GOOGLE_API_KEY=... python scripts/smoke_structured_output.py google
```

## Architecture

TradingAgents is a **LangGraph state machine** that walks a trade through a fixed pipeline of LLM-powered roles. The orchestrator is `tradingagents/graph/trading_graph.py`; the graph topology is built in `tradingagents/graph/setup.py`.

### Pipeline flow (one `propagate(ticker, date)` call)
```
START
  → [selected analysts in sequence: Market → Social → News → Fundamentals]
       each analyst: agent → optional ToolNode → loop until done → Msg Clear
  → Bull Researcher ⇄ Bear Researcher (debate, max_debate_rounds)
  → Research Manager           (structured output: ResearchPlan)
  → Trader                     (structured output: TraderProposal)
  → Aggressive ⇄ Conservative ⇄ Neutral (risk debate, max_risk_discuss_rounds)
  → Portfolio Manager          (structured output: PortfolioRating)
  → END → SignalProcessor extracts rating from rendered markdown (no LLM call)
```

`AgentState` (`tradingagents/agents/utils/agent_states.py`) is a `MessagesState` subclass that carries all per-section reports plus two nested `TypedDict`s (`InvestDebateState`, `RiskDebateState`). Every node returns a partial state update.

### LLM provider abstraction
`tradingagents/llm_clients/factory.py` returns a `BaseLLMClient` instance based on `config["llm_provider"]`. Provider modules are imported lazily so test collection never pulls in heavy SDKs.

- OpenAI-compatible providers (`openai`, `xai`, `deepseek`, `qwen`, `glm`, `ollama`, `openrouter`) → `openai_client.py`.
- `anthropic`, `google`, `azure` have dedicated clients.
- `claude_cli` / `codex_cli` shell out to the locally installed CLI via `cli_client.py` and reuse that CLI's existing auth — no API key needed.

`tradingagents/llm_clients/model_catalog.py` is the single source of truth for which models the CLI offers per provider, in `quick` and `deep` modes. The framework runs two LLMs: `quick_think_llm` (analysts, debaters, trader) and `deep_think_llm` (research manager, portfolio manager).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aiunlocked1412/TradingAgentsGank](https://github.com/aiunlocked1412/TradingAgentsGank) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
