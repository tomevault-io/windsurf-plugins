---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This repo contains one skill covering nine capabilities:
1. **Blave** — Agent calls the Blave REST API directly for crypto market alpha data
2. **BitMart Futures** — Agent calls the BitMart API for perpetual futures trading
3. **BitMart Spot** — Agent calls the BitMart API for spot trading
4. **Bybit** — Agent calls the Bybit API for spot and derivatives/perpetual swap trading
5. **BingX** — Agent calls the BingX API for spot and perpetual swap trading
6. **Bitget** — Agent calls the Bitget API for spot and futures trading
7. **Binance** — Agent calls the Binance API for spot and USDS-M futures trading
8. **Bitfinex** — Agent calls the Bitfinex API for spot, margin, and funding/lending
9. **TWSE / TPEX（台股）** — Agent queries Taiwan stock market data (stock code lookup, quotes, PE/yield/PB) via public APIs; no API key required
10. **TWSE BSR 分點資料** — Agent queries broker/dealer daily trading report via CAPTCHA-protected form; agent solves CAPTCHA using its own vision

No CLI or wrapper involved. All API calls are made directly by the agent.

## Required `.env` Variables

- `blave_api_key`, `blave_secret_key` — Blave API auth
- `BITMART_API_KEY`, `BITMART_API_SECRET`, `BITMART_API_MEMO` — BitMart API auth
- `OKX_API_KEY`, `OKX_SECRET_KEY`, `OKX_PASSPHRASE` — OKX API auth
- `BYBIT_API_KEY`, `BYBIT_API_SECRET` — Bybit API auth
- `BINGX_API_KEY`, `BINGX_SECRET_KEY` — BingX API auth
- `BITGET_API_KEY`, `BITGET_SECRET_KEY`, `BITGET_PASSPHRASE` — Bitget API auth
- `BINANCE_API_KEY`, `BINANCE_SECRET_KEY` — Binance API auth
- `BITFINEX_API_KEY`, `BITFINEX_API_SECRET` — Bitfinex API auth

## Files

| File | Purpose |
|---|---|
| `SKILL.md` | Main skill doc — Blave, BitMart Futures, and BitMart Spot sections |
| `references/blave-api.md` | Blave Python examples |
| `references/blave-indicator-guide.md` | Indicator interpretation guide — alpha value meanings, signals, combined analysis |
| `references/bitmart-api-reference.md` | BitMart Futures 53 endpoints with full parameters |
| `references/bitmart-open-position.md` | Futures open position workflow |
| `references/bitmart-close-position.md` | Futures close position workflow |
| `references/bitmart-plan-order.md` | Futures plan order workflow |
| `references/bitmart-tp-sl.md` | Futures TP/SL workflow |
| `references/bitmart-spot-api-reference.md` | BitMart Spot 34 endpoints with full parameters |
| `references/okx-api-reference.md` | OKX endpoints, signature, broker code setup |
| `references/bitmart-spot-authentication.md` | Spot auth details and examples |
| `references/bitmart-spot-scenarios.md` | Spot common trading scenarios |
| `references/bitmart-signature.md` | Python HMAC-SHA256 signature implementation + common mistakes |
| `references/hyperliquid-api.md` | Hyperliquid API — all 9 endpoints with params, response format, cache times |
| `references/tradingview-stream.md` | TradingView SSE stream — webhook setup, Python streaming client with reconnect |
| `references/bingx-api-reference.md` | BingX 59 endpoints, Python signature, spot + perpetual swap |
| `references/bitget-api-reference.md` | Bitget spot + futures endpoints, Python signature |
| `references/binance-api-reference.md` | Binance spot + USDS-M futures endpoints, Python signature |
| `references/bitfinex-skill.md` | Bitfinex spot, margin, funding/lending endpoints, HMAC-SHA384 signature |
| `references/twse-skill.md` | TWSE/TPEX 台股查詢 — 快速參考：endpoints、欄位說明、Python 搜尋範例 |
| `references/twse-api-reference.md` | TWSE/TPEX 完整 API 參考：上市/上櫃清單、行情、停復牌、民國年轉換 |
| `references/twse-bsr-reference.md` | TWSE BSR 分點資料 — 表單結構、CAPTCHA vision 解碼流程、Python 範例 |

## Blave API Endpoints

Base URL: `https://api.blave.org`

- `price` — current price + 24h change for a symbol (`symbol` required)
- `alpha_table` — latest alpha for all symbols; use for multi-coin queries or screening
- `kline` — OHLCV candlestick data
- `market_direction/get_alpha` — 市場方向 Market Direction (BTCUSDT)
- `market_sentiment/get_symbols` / `get_alpha` — 市場情緒 Market Sentiment time series + stat
- `capital_shortage/get_alpha` — 資金稀缺 Capital Shortage (market-wide)
- `sector_rotation/get_history_data` — 板塊輪動 Sector Rotation history
- `holder_concentration/get_symbols` / `get_alpha` — 籌碼集中度 Holder Concentration time series + stat
- `taker_intensity/get_symbols` / `get_alpha` — 多空力道 Taker Intensity time series + stat
- `whale_hunter/get_symbols` / `get_alpha` — 巨鯨警報 Whale Hunter; supports `score_type`
- `squeeze_momentum/get_symbols` / `get_alpha` — 擠壓動能 Squeeze Momentum + scolor; period fixed to `1d`
- `blave_top_trader/get_exposure` — Blave頂尖交易員 Top Trader Exposure (BTCUSDT)
- `liquidation/get_symbols` — list of symbols with liquidation data
- `liquidation/get_alpha` — 爆倉指標 Liquidation alpha time series + stat; `timeframe` default `24h`
- `liquidation/get_map` — liquidation heatmap: price levels vs USD exposure (`labels`, `liquidation`, `cumsum`, `oi_value`, `price`)
- `liquidation/get_map_change` — recent liquidation events by time window (`hist_0_1h`, `hist_1_8h`, `hist_8_24h`)
- `screener/get_saved_conditions` — user's saved screener conditions

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Blave-TW/blave-quant-skill](https://github.com/Blave-TW/blave-quant-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-09 -->
