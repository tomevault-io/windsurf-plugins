---
trigger: always_on
description: Tkinter desktop app that monitors intraday P&L across a UBS brokerage and UBS 401k account. Holdings come from `claudedev_shared`; prices come from Yahoo Finance via `yfinance`. Charts redraw on demand or on a 60 s auto-update loop.
---

# PnL-Monitor

## Overview
Tkinter desktop app that monitors intraday P&L across a UBS brokerage and UBS 401k account. Holdings come from `claudedev_shared`; prices come from Yahoo Finance via `yfinance`. Charts redraw on demand or on a 60 s auto-update loop.

## Environment
- **Conda env:** `PnL-Monitor` — interpreter `C:\Users\wamfo\anaconda3\envs\PnL-Monitor\python.exe`. Never use system Python.
- **Deps:** `environment.yml` (Python 3.13). pip pins are `>=X.Y,<X+1.0`: `yfinance`, `pandas`, `matplotlib`, `squarify`, `sv-ttk`, `claudedev_shared`, `pytest`.
- **Launch:** `main.py` (configures logging + `sv_ttk` light theme + `PnLApp`). `Launch PnL Monitor.bat` uses `pythonw.exe` for console-less double-click launch. User docs in `MANUAL.md`.
- **GitHub:** https://github.com/marange63/PnL-Monitor (branch `main`).

## Data Pipeline
- `ubs_live_price_holdings()` + `ubs_401k_holdings()` return DataFrames with `DESCRIPTION`, `SYMBOL`, `SOD VALUE`, `Ticker Alias`, `Tag`, `Source`.
- Prices via `yf.Ticker(t).fast_info` — use `last_price` and `regular_market_previous_close` (**not** `previous_close`, which includes after-hours).
- `load_and_compute()` concatenates both sources, fetches prices in parallel (`ThreadPoolExecutor(max_workers=5)`, one retry with 0.5 s backoff, failed tickers yield `(None, None, None)`), then fills `Last Price`, `Last Close`, `% Move On Day`, `PnL`.
- **Split auto-detect** (`_detect_split_ratio`): if `last_price / last_close < 0.35`, divide `last_close` by `round(1/ratio)`. Logs `detected N:1 split` INFO line — check the log before chasing "too good" prices.

## DataFrame Schema
Reference columns through `constants.Col` — never string literals.

| `Col.` attr | Column | Notes |
|---|---|---|
| `DESCRIPTION` | `DESCRIPTION` | Security full name |
| `SYMBOL` | `SYMBOL` | Brokerage symbol |
| `SOD_VALUE` | `SOD VALUE` | Start-of-day USD |
| `TICKER` | `Ticker Alias` | Yahoo ticker |
| `SOURCE` | `Source` | `"UBS"` or `"401K"` |
| `TAG` | `Tag` | Sector/category |
| `LAST_PRICE` | `Last Price` | from yfinance |
| `LAST_CLOSE` | `Last Close` | split-adjusted if detected |
| `PCT_MOVE` | `% Move On Day` | **decimal**, e.g. `0.0179` = 1.79% |
| `PNL` | `PnL` | `SOD VALUE * PCT_MOVE` |

## Modules
| File | Role |
|---|---|
| `main.py` | Entry point: logging, theme, `PnLApp` |
| `app.py` | `PnLApp` — wires control strip, panes, and `RunLoop`; owns toggle vars; persists custom tickers to `custom_tickers.json` (`MAX_CUSTOM_TICKERS = 4`) |
| `run_loop.py` | `RunLoop` — worker thread + auto-update timer + countdown tick; calls `on_result(RunResult)` on main thread |
| `data.py` | `load_and_compute`, `get_price_data`, `validate_ticker`, `get_default_drawdowns`, `get_drawdowns`, `get_intraday_prices`, `HighCache`; `DEFAULT_TICKERS`, `INTRADAY_TICKERS` |
| `charts.py` | `draw_scatter`/`draw_bar`/`draw_treemap` + their `build_*_df` companions; `dollar_fmt`, `pct_fmt` |
| `constants.py` | `Col`, color palette, button colors, sizing/timing constants |
| `drawdown_table.py` | `DrawdownTable` — ticker/6W/ATH table (+ optional Today column via `show_today`); editable variant adds × delete labels, Add row, async validation, height-sync to sibling |
| `bar_chart_pane.py` | `ScrollableBarChart` — debounced resize, mousewheel scroll, hover; used for both ticker bar and tag bar |
| `intraday_panel.py` | `IntradayChartGrid` — row of % return vs prev-close thumbnails, shared y-range, segment-colored `LineCollection` |
| `scatter_pane.py` | `ScatterPane` — SOD vs PnL/Return, hover + log-X toggle |
| `treemap_pane.py` | `TreemapPane` — \|PnL\|-sized tiles, sign-colored, 150 ms debounced resize |
| `tooltip.py` | `Tooltip` — singleton yellow floater shared by every pane |

## GUI Layout

### Control strip (row 0)
Col 0 **ETF % from Highs** (SPY/QQQ/IWM/EEM) · Col 1 **Custom % from Highs** (up to 4 user tickers with × delete + Add row that only appears when count < 4; shows extra **Today** column) · Col 2 buttons + status + PnL summary · Col 3 **Intraday** thumbnails (SPY/QQQ/SMH, 1-min `period="1d"`).

Buttons: **Run** · **Auto Update** (60 s loop; label becomes `Stop (Ns)` with live countdown) · **Log X** · **Group Tickers** · **Return %** · **Sort A–Z** · **Export CSV**. PnL summary shows UBS / 401K / Total, green/red by sign.

Intraday y-axis is **% return vs prev close** with a shared y-range across all three charts. Each chart is a `LineCollection` whose segments are colored by sign of midpoint; dotted gray 0% line; title color tracks sign of latest value.

ETF and Custom tables share a single `HighCache` (highs cached per calendar day; `bump`ed up if `last_price` overshoots). Custom pane's height is pinned to ETF's via `grid_propagate(False)` so they stay aligned.

### Keyboard
`F5` Run · `Ctrl+A` toggle Auto Update · `Ctrl+S` Export CSV

### Panes (row 1, `ttk.PanedWindow`, horizontal)
Order and initial weights: **treemap (7) | scatter (9) | ticker bar (6) | tag bar (7)**.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [marange63/PnL-Monitor](https://github.com/marange63/PnL-Monitor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
