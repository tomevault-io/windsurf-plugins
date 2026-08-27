---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

US stock real-time monitoring, backtesting, and portfolio analysis application built with Python/Tkinter. Uses Yahoo Finance (yfinance) for market data. UI and comments are in Korean.

## Commands

```bash
# Install dependencies
pip install -r requirements.txt

# Run the application
python stock_monitor_gui.py

# Build Windows executable
pyinstaller stock_monitor_gui.spec
```

There are no tests or linting configured.

## Architecture

```
stock_monitor_gui.py              # Main Tkinter GUI, entry point (AppState singleton)
  └── modules/                    # All supporting modules (added to sys.path at startup)
      ├── __init__.py
      ├── stock_score.py          # Technical analysis (RSI, MA, MACD, Bollinger, momentum, Ichimoku)
      ├── market_trend_manager.py # US market session detection, trend caching, volatility regime
      ├── config.py               # JSON config loading with recursive default merging, atomic writes
      ├── backtest_popup.py       # Strategy backtesting engine + matplotlib charts (8 strategies)
      ├── news_panel.py           # Finviz news scraping, sentiment classification, ticker linking
      ├── data_cache.py           # SQLite cache for yfinance data (delta updates, TTL-based expiry)
      ├── pattern_recognition.py  # Chart pattern detection (double top/bottom, H&S, triangles) via scipy
      ├── fundamental_score.py    # Valuation scoring, Piotroski F-Score, factor scoring
      ├── portfolio_analysis.py   # Correlation, optimization (4 methods + efficient frontier), Black-Litterman, Fama-French
      ├── portfolio_backtest.py   # Portfolio-level backtest: monthly momentum top-N rebalancing vs SPY (분석 menu)
      ├── holdings_manager.py     # Portfolio holdings CRUD (holdings.json), position/P&L calculation
      ├── quant_screener.py       # Quantitative screening (7 strategies: buffett/graham/lynch/greenblatt/dividend/momentum/multifactor)
      ├── screener_popup.py       # Screener UI popup with Treeview results + detail panel
      ├── stock_universe.py       # Stock universe providers (S&P500/NASDAQ100/DOW30 bundled + online + CSV) + search_symbols() (ticker/company-name lookup via yf.Search with HTTP fallback)
      ├── help_texts.py           # Centralized Korean help/tooltip strings
      ├── ui_components.py        # Reusable Tooltip / HelpTooltip widgets (theme-aware)
      ├── theme.py                # Dark/light theme system (ttk 'clam' styles, tk option_add defaults, matplotlib rcParams)
      ├── price_alerts.py         # Price alert system (alerts.json, threshold checks, toast notifications, manager dialog)
      ├── korean_aliases.py       # Korean stock-name alias dictionary (korean_aliases.json, seeded from built-in defaults; manager dialog; powers Korean search + table name suffix)
      ├── stock_memos.py          # Per-ticker memos (stock_memos.json, multiline editor dialog; 📝 marker in table)
      └── file_io.py              # Shared atomic JSON write (tempfile+fsync+os.replace), corrupt-file backup, and backup_files() (move files to a backup dir); used by all JSON persistence paths + the 데이터 초기화 feature
```

`stock_monitor_gui.py` adds `modules/` to `sys.path` at startup, so all inter-module imports (`import config`, `from fundamental_score import ...`) work unchanged.

### Data Flow

GUI spawns a daemon thread (`monitor_stocks`) that refreshes every 60 seconds. Each refresh uses `ThreadPoolExecutor(max_workers=10)` to call `fetch_stock_data()` in parallel for all tickers in `watchlist.json`. Rows update incrementally as each ticker completes (`upsert_record_row()` via `root.after`); `app._row_by_ticker` maps tickers to Treeview rows and `app.all_records` keeps the latest record per ticker (used by the toolbar search filter). On startup, the last refresh results are restored instantly from `table_snapshot.json` before fresh data arrives. `refresh_table_once()` is guarded against re-entry by `app._refresh_guard`.

`fetch_stock_data()` minimizes slow yfinance calls: live quotes come from `fast_info` and fundamentals from the 24h SQLite `fundamental_cache` (`_get_quote_bundle()`); the full `.info` scrape only runs during pre/after-market sessions (pre/post prices exist only there). Earnings dates are cached 12h in `misc_cache`; higher-timeframe trend is cached in-memory for 10 minutes.

### Backtesting

Double-clicking a ticker row (or clicking a ticker in the news panel) opens `backtest_popup.py`, which runs one of 8 strategies (macd, rsi, bollinger, ma_cross, macd+rsi, momentum_signal, momentum_return_ma, ichimoku) against historical data and renders matplotlib charts with buy/sell markers. Includes strategy comparison and sensitivity analysis embedded in the result container.

### Configuration

`config.py` loads `config.json` with recursive merging against defaults. Three presets (short/middle/long) control period, interval, and indicator parameters. Access via `config.config` proxy (lazy-loaded, thread-safe). `get_risk_free_rate()` fetches ^TNX with 1hr cache, 4.5% fallback.

### Data Caching


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LEE-YANG-JAE/stock_monitor](https://github.com/LEE-YANG-JAE/stock_monitor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-27 -->
