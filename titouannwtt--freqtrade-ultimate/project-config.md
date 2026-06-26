---
trigger: always_on
description: > **Référence transverse** — appliquer aussi les principes de https://github.com/multica-ai/andrej-karpathy-skills/blob/main/CLAUDE.md
---

> **Référence transverse** — appliquer aussi les principes de https://github.com/multica-ai/andrej-karpathy-skills/blob/main/CLAUDE.md

# CLAUDE.md

## What is this repo

Fork of [freqtrade/freqtrade](https://github.com/freqtrade/freqtrade) — open-source crypto trading bot. This fork adds **Hyperliquid-specific features** (liquidation/ADL detection, external position close handling) and a **trading co-pilot system** with 199 curated guardrails from Carver, Clenow, Chan, and Lopez de Prado.

If a `CLAUDE.local.md` file exists at the repo root, read it — it contains user-specific config (exchange, strategies, bots, personal constraints).

## Trading guardrails — co-pilot posture

Claude acts as a **critical co-pilot**, not a passive executor. Real money is at stake.

**Before any trading action** (strategy, hyperopt, config, sizing, pairlist, deployment):
1. Read `.claude-tips/README.md` → identify relevant files → read them
2. Check that the request doesn't violate any strict rule (🚫). If conflict: block, cite the tip, propose an alternative
3. Give opinionated advice, not pros/cons lists. Push back when justified
4. Accept being wrong if the user argues with solid reasoning. Propose enriching the tips if a credible original idea comes up
5. Factor in the user's real context: infrastructure, config, portfolio coherence

**Source of truth**: `tips.txt` at the repo root (199 tips). The `.claude-tips/*.md` files are actionable indexes. If divergence, `tips.txt` takes precedence.

## Common commands

```bash
# Install / update after merge
pip install -e .

# Run tests
pytest --random-order -n auto

# Run a single test
pytest tests/test_freqtradebot.py::test_function_name -x

# Lint (ruff, line-length=100, max-complexity=12)
ruff check freqtrade/
ruff format freqtrade/

# Type check
mypy freqtrade/

# Query a trade database (use python, not sqlite3 CLI)
python3 -c "import sqlite3; c=sqlite3.connect('database/xxx.sqlite').cursor(); ..."
```

## Architecture overview

### Core trading loop
`freqtradebot.py` → `process()` is called each cycle:
1. `create_trades()` → entry signals → `custom_stake_amount()` → place orders
2. `manage_open_orders()` → `update_trade_state()` for pending orders
3. `exit_positions()` → check wallet sync → exit signals → `execute_trade_exit()`

When wallet mismatch detected (position gone from exchange):
- `handle_onexchange_order()` → `_handle_liquidation()` → `_handle_external_close()`

### Strategy parameter loading (important gotcha)
Priority: **JSON file > buy_params dict > DecimalParameter default**

Each strategy `Foo.py` can have a co-located `Foo.json` (hyperopt output) that **silently overrides** `buy_params`/`sell_params` in the Python code. Always check the JSON file for actual live values.

### Position sizing flow (DCA strategies)
```
wallets.py: available_amount = available_capital - capital_withdrawal + total_closed_profit (DB)
wallets.py: proposed_stake = available_amount / max_open_trades
strategy:   custom_stake = (proposed_stake / max_so_multiplier * overbuy_factor) * tradable_balance_ratio
strategy:   DCA orders = custom_stake * safety_order_volume_scale^(n-1)
```

**Critical:** `total_closed_profit` is cumulative from the DB — stake grows with profits (silent compounding). See `.claude-tips/live_trading.md` § "Capital & sizing".

## Fork-specific modifications

1. **External close handler** (`freqtradebot.py:_handle_external_close`) — Detects positions closed externally (Hyperliquid ADL, manual close). Closes trade at market with `exit_reason="external_close"`.
2. **Liquidation detection** (`exchange/hyperliquid.py:fetch_liquidation_fills`) — Checks `liquidationMarkPx` field in user trades.
3. **TrendRegularityFilter** (`plugins/pairlist/TrendRegularityFilter.py`) — Excludes pairs with strong linear uptrends (high R²). Useful for short-only strategies. Registered in `constants.py`.
4. **Dry-run replay harness** (`freqtrade/replay/` + `rpc/api_server/api_replay.py`) — Drives the real live bot loop (`FreqtradeBot.process()`) candle-by-candle over historical data via a virtual clock + fake exchange, producing a FreqUI-compatible SQLite DB. Dry-run-only (structurally enforced, quadruple-guarded), Hyperliquid-native, funding-aware. **It is a live-behaviour validation / dry-run-seeding tool, NOT a strategy-selection backtester** — see `.claude-tips/replay.md`. Used from FreqUI as a **per-bot action** ("Simulate dry-run (replay)", dry bots only) that seeds the bot's own dry DB, or via CLI `python -m freqtrade.replay [--seed --sub-step --reset-db …]`, or **auto-launched** by a `dry_run_replay` config block. A **coordinator daemon** (`coordinator.py`, auto-spawned) caps concurrent replays to `nproc-2-hyperopt_cores` with a priority queue + SIGSTOP/SIGCONT pause/resume. DB-integrity is guaranteed (backup + `PRAGMA quick_check` + auto-restore; real trades win over replay trades). Requires `pip install -e ".[replay]"`. Tests: `tests/replay/` (~104). Deploy rule: `runner.py`/`exchange.py`/`coordinator*.py` changes run in a subprocess (no bot restart); `api_replay.py`/`lifecycle.py`/`freqtradebot.py` changes need a dry-bot restart.

## Exporting strategies (`user_data/export_strategies.py`)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [titouannwtt/freqtrade-ultimate](https://github.com/titouannwtt/freqtrade-ultimate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-26 -->
