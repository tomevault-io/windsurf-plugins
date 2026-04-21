---
trigger: always_on
description: - **Package manager:** `uv` — always use `uv run`, `uv add`, `uv sync` (never pip)
---

# Claude Code Context — CryptoApp

## Developer Preferences

- **Package manager:** `uv` — always use `uv run`, `uv add`, `uv sync` (never pip)
- **No emojis:** Never use emoji characters anywhere — not in JS strings, HTML templates, Python log messages, or route responses. Use plain text instead (e.g. `STOPPED` not `🛑 STOPPED`, `Auto-approving` not `🤖 Auto-approving`).
- **Code style:** Keep the project tidy — no redundant files, dead code, or orphaned imports
- **Git workflow:** `dev` branch for work, merge to `main` for releases. Commit + push after every completed change. Descriptive commit messages with bullet-point body.
- **Deployment:** Raspberry Pi 4 — if running locally use `ssh pi "cd ~/CryptoApp && git pull && sudo systemctl restart cryptoapp"`. If already on the Pi, run `git pull && sudo systemctl restart cryptoapp` directly.

## Verification Commands

After making changes, verify with:
- **Syntax check:** `uv run python -m py_compile <file>.py` (also runs automatically via hook on every edit)
- **Import check:** `uv run python -c "import app"` — catches missing deps and circular imports
- **Run app locally:** `uv run python app.py` (dev mode, port 5001)
- **Health check:** `curl -s http://localhost:5001/` — should return 200
- **Lint:** `uv run flake8 ml/ routes/ --max-line-length=120 --ignore=E501`

## Context Compaction

When compacting, ALWAYS preserve:
- The list of files modified in this session and what changed
- Any in-progress task or feature being built
- Current error messages or test failures being debugged
- Trade safety mechanism states (kill switch, budget, approval settings)

## Project Overview

AI-powered low-cap cryptocurrency analysis and automated trading system. Uses a 5-agent Google ADK (Gemini) architecture to discover undervalued coins, analyse them, and execute live trades on Kraken. Runs headlessly on a Raspberry Pi 4 for ~£2.50/month.

**Key flow:** CoinGecko data → 5 AI agents analyse → trading engine proposes → auto-approve or email → Kraken execution → portfolio tracking → sell automation monitors exits.

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Language | Python 3.13 |
| Web | Flask + Gunicorn (1 worker, 4 gthread) |
| AI Agents | Google ADK + `gemini-2.0-flash` (configurable via `ORCHESTRATOR_MODEL`) |
| Exchange | Kraken via ccxt |
| ML | scikit-learn (training), ONNX Runtime (inference), Q-learning RL |
| Frontend | Vanilla JS + Jinja2 templates + CSS |
| Data | JSON file persistence, optional Redis cache |
| Deploy | systemd + nginx + Let's Encrypt SSL |
| Scheduling | `schedule` library in background threads |

## Directory Structure

```
app.py              — Flask app factory, extension init, scheduler starts
wsgi.py             — Gunicorn WSGI entry (wsgi:app)
gunicorn.conf.py    — Pi-optimised config (1 worker, 120s timeout)
extensions.py       — Flask-Limiter, Flask-CORS (avoids circular imports)

ml/                 — All ML, trading, and agent logic
  scan_loop.py      — Automated scan pipeline (every 12h)
  market_monitor.py — Between-scan price checks at 5, 15, and 30-min intervals
  trading_engine.py — Proposals, budget, approval, execution, kill switch
  exchange_manager.py — Kraken via ccxt, pair cache, FX, orders
  sell_automation.py  — Exit triggers (profit/stop-loss/trailing/agent)
  q_learning.py        — Q-learning RL (buy/skip, reward shaping, ε-greedy)
  portfolio_tracker.py — Holdings, cost basis, unrealised P&L
  portfolio_manager.py — Batch portfolio analysis via ADK
  orchestrator_wrapper.py — Thin ADK adapter for portfolio batch analysis
  training_pipeline.py — RandomForest → ONNX export
  onnx_inference.py    — ONNX Runtime inference
  backtesting.py       — Backtesting framework
  data_pipeline.py     — Data ingestion and feature engineering
  gem_score_tracker.py — Tracks gem scores over time
  error_handling.py    — Shared error handling utilities
  agent_memory.py      — Short/long-term agent context
  scheduler.py         — Weekly retrain + performance reports
  doc_updater.py       — Auto-updates documentation
  agents/official/     — 5 ADK agents + orchestrator + quick_screen

routes/             — Flask Blueprints (coins, health, ml, symbols, trading)
services/           — Shared app state + optional Redis cache
src/core/           — Config, CryptoAnalyzer model, CoinGecko fetcher
src/web/            — Jinja2 templates + static JS/CSS
data/               — Runtime JSON state (gitignored)
deploy/             — systemd unit, nginx config, SSL script
docs/               — Markdown documentation
```

## Key Patterns

- **Singletons:** `get_trading_engine()`, `get_scan_loop()`, `get_exchange_manager()` — module-level globals with lazy init
- **State init:** `services/app_state.py` → `init_all()` called once at startup
- **Error handling:** try/except with `logger.warning()` fallback; graceful degradation if dependencies missing
- **Logging:** Per-module `logger = logging.getLogger(__name__)`
- **Data models:** `@dataclass` for core models, Pydantic `BaseModel` for agent I/O
- **Section separators:** `# ─── Section Name ───────────────────`
- **Auth:** Bearer token (`TRADING_API_KEY`) on all trading POST endpoints

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/finn6666) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
