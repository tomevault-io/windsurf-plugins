---
trigger: always_on
description: This document is written for an AI coding agent (Claude Sonnet 4.6) working on this
---

# AGENTS.md — Polymarket BTC Backtesting Tool

This document is written for an AI coding agent (Claude Sonnet 4.6) working on this
codebase. Read this before touching any file.

---

## Project Purpose

Backtest trading strategies on Polymarket BTC "Up or Down" 5-minute and 15-minute
prediction markets using real historical CLOB order-book data.

The tool does NOT trade live. It replays recorded market episodes, feeds each
~100 ms snapshot to a strategy, simulates realistic order fills against the real
CLOB event stream, and records every trade in full detail for analysis via a
Streamlit UI.

---

## Data Source

**Dataset**: `trentmkelly/polymarket_crypto_derivatives` (HuggingFace)
**Coverage**: 2026-02-21 → 2026-03-24  
**Size**: ~32.3 GB  
**Symbols**: BTC, ETH, SOL, XRP — but we filter to BTC only.  
**Intervals**: 5-minute and 15-minute markets.

### Episode Directory Layout

Each market episode is one directory on HuggingFace:

```
btc5m_market<id>_<window_start_unix_s>_all/
  steps.parquet        # one row per ~100ms decision snapshot
  events.parquet       # CLOB events attached to the following step
  book_levels.parquet  # full orderbook depth per step
```

### steps.parquet Schema

| Column | Type | Description |
|---|---|---|
| step_index | int | monotonically increasing within episode |
| ts | int | Unix milliseconds |
| progress | float | fraction of market lifetime elapsed (0 → 1) |
| dt_s | float | seconds since previous step |
| hour | int | UTC hour |
| chainlink_price | float | BTC/USD from Chainlink (this is the resolution oracle) |
| binance_price | float | BTC spot from Binance |
| up_best_bid | float | CLOB best bid for UP token (0–1) |
| up_best_ask | float | CLOB best ask for UP token (0–1) |
| up_mid | float | (best_bid + best_ask) / 2 for UP |
| up_spread | float | best_ask - best_bid for UP |
| up_bid_size_total | float | total USD on bid side for UP |
| up_ask_size_total | float | total USD on ask side for UP |
| up_imbalance | float | (bid_size - ask_size) / (bid_size + ask_size) for UP |
| down_best_bid | float | same fields for DOWN token |
| down_best_ask | float | |
| down_mid | float | |
| down_spread | float | |
| down_bid_size_total | float | |
| down_ask_size_total | float | |
| down_imbalance | float | |

### events.parquet Schema

| Column | Type | Description |
|---|---|---|
| following_step_index | int | these events occurred before this step |
| event_index | int | order within step |
| event_type | int | 1=trade, 2=price/book update, 3=tick size change |
| ts | int | Unix milliseconds |
| is_down | bool | True = event is for DOWN token, False = UP |
| is_sell | bool | True = sell order |
| is_sell_side | bool | True = from sell side perspective |
| price | float | Polymarket contract price (0–1) |
| size | float | CLOB token size |
| old_tick_size | float | (type 3 only) |
| new_tick_size | float | (type 3 only) |

### book_levels.parquet Schema

| Column | Type | Description |
|---|---|---|
| step_index | int | matches steps.parquet step_index |
| outcome | int | 0=UP, 1=DOWN |
| side | int | 0=bid, 1=ask |
| level_index | int | 0=best, 1=second-best, ... |
| price | float | price level (0–1) |
| size | float | total size at this price level |

---

## Architecture Overview

```
HuggingFace Dataset
        │
        ▼
data/downloader.py     ── downloads & caches parquet files locally
        │
        ▼
data/loader.py         ── DuckDB queries → assembles MarketEpisode objects
        │
data/filter.py         ── filters episodes list by symbol/interval/date
        │
        ▼
engine/backtester.py   ── main loop
    for each episode:
        strategy.reset()
        for each step (chronological):
            book = build BookSnapshot from step row
            events = events for this step
            order = strategy.on_step(step, book, events, position)
            if order and position is None:
                fill = fill_simulator.try_fill(order, step, future_events)
                if fill: create ActivePosition
        on resolution:
            close position → create TradeRecord
        │
        ▼
analytics/run_result.py  ── collects TradeRecords → BacktestResult → saves JSON
        │
        ▼
ui/app.py  (Streamlit)   ── loads run JSONs → 3-page UI
```

---

## Data Layer (`data/`)

### `models.py`

Defines all dataclasses used across the system. DO NOT add business logic here.

- `Step`: one row from steps.parquet, all fields as Python floats/ints
- `BookSnapshot`: wraps UP and DOWN `SideSnapshot` objects (convenient access)
- `SideSnapshot`: best_bid, best_ask, mid, spread, imbalance, bid_size_total, ask_size_total
- `CLOBEvent`: one row from events.parquet
- `BookLevel`: one row from book_levels.parquet
- `MarketEpisode`: market_id, interval, window_start_ts, steps[], events[], book_levels[], resolution

### `downloader.py`

Downloads episodes from HuggingFace Hub into a local `./cache/` directory.
Supports partial download (skip already-cached episodes).
Exposes `list_btc_episodes()` → list of episode directory names.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tonio0/test](https://github.com/tonio0/test) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-27 -->
