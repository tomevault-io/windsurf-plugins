---
trigger: always_on
description: > The complete reference for AI agents operating pftui as their financial data layer.
---

# AGENTS.md — Agent Operator Guide

> The complete reference for AI agents operating pftui as their financial data layer.
>
> **First time?** Start with [ONBOARDING.md](ONBOARDING.md) — it walks through installation, portfolio setup, and the first week of operation.
>
> This file covers: analytics engine, CLI reference, data model, integration patterns, multi-timeframe agent architecture, and best practices.
>
> For code contribution, see [CLAUDE.md](CLAUDE.md).
> For architecture reference, see [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md).
> For AI operating model details, see [docs/AI-LAYER.md](docs/AI-LAYER.md).
> For always-on deployment, see [docs/DAEMON.md](docs/DAEMON.md).

---

## Table of Contents

1. [Analytics Engine](#analytics-engine)
2. [CLI Reference](#cli-reference)
3. [Data Model](#data-model)
4. [Integration Patterns](#integration-patterns)
5. [Multi-Timeframe Agent Architecture](#multi-timeframe-agent-architecture-advanced)
6. [Best Practices](#best-practices)

---

## Analytics Engine

pftui's core is a multi-timeframe analytics engine operating across four layers:
LOW (hours→days), MEDIUM (weeks→months), HIGH (months→years), MACRO (years→decades).
Each layer uses different data, updates at different frequencies, and produces different signals.
Layers constrain downward and signal upward. Use `pftui analytics signals` for active cross-timeframe signals.

### Scenarios (`pftui journal scenario`)
Track macro scenarios with probability estimates. Each probability update is logged
to history for calibration. Signals track evidence for/against each scenario.

### Thesis
Thesis tracking is maintained as narrative workflow files (`THESIS.md`) and journal notes.

### Convictions (`pftui journal conviction`)
Asset-level conviction scores (-5 to +5) over time. Append-only log — every
`set` creates a new row. Current conviction = latest row per symbol.
For negative scores, use `--score=-2`.

### Agent Signals (`pftui analytics signals`)
Cross-timeframe signal detection (alignment/divergence/transition) computed during
`pftui data refresh` and stored in `timeframe_signals`.

---

## CLI Reference

### Portfolio State

| Command | What It Returns |
|---|---|
| `pftui portfolio brief --json` | Complete portfolio snapshot — positions, allocations, movers, technicals, macro |
| `pftui portfolio value --json` | Total value with category breakdown and daily change |
| `pftui portfolio summary --json` | Detailed position-level data — price, quantity, cost basis, gain/loss, allocation % |
| `pftui portfolio performance --json` | Returns: 1D, MTD, QTD, YTD, since inception |
| `pftui portfolio drift --json` | Current vs target allocation with drift % and rebalance suggestions |
| `pftui portfolio history --date YYYY-MM-DD --json` | Historical portfolio snapshot for any past date |
| `pftui system export json` | Full portfolio export (positions + transactions) |
| `pftui portfolio transaction list` | List all transactions with IDs |

### Market Data

| Command | What It Returns |
|---|---|
| `pftui data refresh` | Fetches ALL data sources (10+ sources, ~50 symbols) |
| `pftui data dashboard macro --json` | DXY, VIX, yields, currencies, commodities, derived ratios |
| `pftui data fear-greed --json` | Latest crypto + traditional Fear & Greed readings with optional history |
| `pftui portfolio watchlist --json` | All watched symbols with prices, day change, 52W range |
| `pftui analytics movers --json [--threshold N] [--overnight]` | Significant daily/overnight moves (default >3%) |
| `pftui data predictions --json [--limit N]` | Polymarket prediction market odds |
| `pftui data sentiment --json` | Crypto + traditional Fear & Greed, COT positioning |
| `pftui data news --json [--limit N]` | Financial news from RSS feeds |
| `pftui data supply --json` | COMEX gold/silver inventory |
| `pftui data dashboard global --json` | World Bank macro data (GDP, debt, reserves) |
| `pftui data status --json` | Data source freshness plus daemon health — last update time per source + `daemon` heartbeat |

### Portfolio Management

| Command | What It Does |
|---|---|
| `pftui portfolio transaction add --symbol SYM --category CAT --tx-type buy/sell --quantity N --price P --date D` | Add transaction |
| `pftui portfolio transaction remove ID` | Remove transaction by ID |
| `pftui portfolio set-cash CURRENCY AMOUNT` | Set cash position |
| `pftui portfolio watchlist add SYMBOL [--target PRICE]` | Add to watchlist |
| `pftui portfolio watchlist remove SYMBOL` | Remove from watchlist |
| `pftui portfolio target set SYMBOL --target PCT` | Set target allocation % |
| `pftui portfolio target remove SYMBOL` | Remove target |
| `pftui portfolio rebalance --json` | Suggested trades to reach targets |
| `pftui portfolio broker add BROKER --api-key KEY [--secret SECRET]` | Connect a broker (trading212, ibkr, binance, kraken, coinbase, crypto-com) |
| `pftui portfolio broker sync [BROKER] [--dry-run] --json` | Sync positions from connected brokers |
| `pftui portfolio broker list --json` | List configured broker connections |
| `pftui portfolio broker remove BROKER` | Remove a broker and its synced transactions |
| `pftui analytics alerts add "CONDITION"` | Add alert |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [skylarsimoncelli/pftui](https://github.com/skylarsimoncelli/pftui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
