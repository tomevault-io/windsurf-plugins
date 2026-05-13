---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a **unified MCP (Model Context Protocol) server** for BIST (Istanbul Stock Exchange), US stocks, cryptocurrencies, mutual funds, and FX data. The server consolidates 81 legacy tools into **27 unified, function-based tools** with market routing.

**⭐ MAJOR CONSOLIDATION (v0.9.0):**
- **81 tools → 27 unified tools** (68% reduction)
- **Market-based routing**: Single tool handles BIST, US, crypto via `market` parameter
- **Multi-ticker parallel execution**: 75% faster batch queries
- **Unified response models**: Consistent data structures across all markets
- **NEW**: News detail support, Islamic finance compliance, fund comparison
- **NEW**: Macro inflation data, screener/scanner help tools

**Key Features:**
- **Stocks (BIST + US)**: Search, profile, quick info, technicals, financials, analysts, dividends, earnings
- **Crypto (BtcTurk + Coinbase)**: Ticker, orderbook, trades, OHLC, technical analysis
- **Funds (TEFAS)**: 836+ Turkish mutual funds with portfolio, performance, and comparison
- **FX (borsapy)**: 65 currencies, precious metals, commodities
- **Macro**: Economic calendar (7 countries), bond yields, inflation data, sector comparisons
- **Help**: Screener presets/filters, scanner indicators/presets, fund regulations

## Architecture

The project follows a **unified router pattern** with market-based routing:

### Core Files
- **unified_mcp_server.py**: Main FastMCP server with 27 unified tools (v0.9.0+)
- **providers/market_router.py**: Market routing layer that dispatches to providers
- **models/unified_base.py**: Unified response models and enums (84 exports)
- **borsa_mcp_server.py**: Legacy server with 81 tools (kept as fallback)

### Provider Layer
- **providers/**: Data provider modules
  - `kap_provider.py`: 758 BIST companies with multi-ticker support
  - `yfinance_provider.py`: Complete financial data (BIST + US)
  - `isyatirim_provider.py`: İş Yatırım financial statements and ratios
  - `tefas_provider.py`: TEFAS mutual fund data provider
  - `btcturk_provider.py`: BtcTurk cryptocurrency provider
  - `coinbase_provider.py`: Coinbase global crypto provider
  - `borsapy_fx_provider.py`: Currency and commodities via borsapy
  - `borsapy_calendar_provider.py`: Economic calendar (TR, US, EU, DE, GB, JP, CN)
  - `borsapy_bond_provider.py`: Turkish government bond yields
  - `borsapy_scanner_provider.py`: BIST technical scanner (TradingView)
  - `yfscreen_provider.py`: US securities screener
  - `buffett_analyzer_provider.py`: Warren Buffett value investing
  - `financial_ratios_provider.py`: Financial ratio calculations

## Key Development Commands

```bash
# Run the unified MCP server (22 tools)
uv run python unified_mcp_server.py
uv run borsa-mcp  # Entry point

# Run legacy server (81 tools) - for backwards compatibility
uv run python borsa_mcp_server.py
uv run borsa-mcp-legacy

# Install dependencies
uv pip install -r requirements.txt

# Build the package
rm -rf build/ && uv build

# Test unified server
uv run python -c "from unified_mcp_server import app; print('Server OK')"

# Test legacy functionality
uv run test_mcp_server.py
uv run test_kap_haberleri.py
uv run test_tefas_provider.py
```

## Complete Tool Interface (26 Unified Tools)

### Stock Tools (15 tools - BIST + US markets)
| Tool | Description | Multi-ticker |
|------|-------------|--------------|
| `search_symbol` | Search stocks, indices, funds, crypto by name/symbol | - |
| `get_profile` | Company profile with sector, description, financials + Islamic finance compliance (BIST) | - |
| `get_quick_info` | Quick metrics (P/E, P/B, ROE, 52w range) | ✅ |
| `get_historical_data` | OHLCV price data with date range support | - |
| `get_technical_analysis` | RSI, MACD, Bollinger Bands, moving averages (BIST, US, crypto) | - |
| `get_pivot_points` | Support/resistance levels (S1-S3, R1-R3) | - |
| `get_analyst_data` | Analyst ratings and price targets | ✅ |
| `get_dividends` | Dividend history, yield, payout ratio | ✅ |
| `get_earnings` | Earnings calendar, EPS history, growth estimates | ✅ |
| `get_financial_statements` | Balance sheet, income statement, cash flow | ✅ |
| `get_financial_ratios` | Valuation, Buffett, health, advanced metrics | - |
| `get_corporate_actions` | Capital increases, dividend history (BIST) | ✅ |
| `get_news` | KAP news list or detail view with news_id (BIST) | - |
| `screen_securities` | Screen with 23 presets or custom filters | - |
| `scan_stocks` | Technical scanner (RSI, MACD, Supertrend, T3) | - |

### Crypto Tools (1 tool - BtcTurk + Coinbase)
| Tool | Description |
|------|-------------|
| `get_crypto_market` | Ticker, orderbook, trades, OHLC, exchange info |

### FX & Macro Tools (5 tools)
| Tool | Description |
|------|-------------|
| `get_fx_data` | 65 currencies, metals, commodities via borsapy |
| `get_economic_calendar` | Economic events (TR, US, EU, DE, GB, JP, CN) |
| `get_bond_yields` | Government bond yields (TR 2Y, 5Y, 10Y) |
| `get_sector_comparison` | Sector peers and average metrics |
| `get_macro_data` | Turkish inflation data (TÜFE/ÜFE) and inflation calculator |


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [saidsurucu/borsa-mcp](https://github.com/saidsurucu/borsa-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
