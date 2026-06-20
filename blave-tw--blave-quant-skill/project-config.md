---
trigger: always_on
description: Use for: (1) Blave market alpha data — 籌碼集中度 Holder Concentration, 多空力道 Taker Intensity, 巨鯨警報 Whale Hunter, 擠壓動能 Squeeze Momentum, 市場方向 Market Direction, 資金稀缺 Capital Shortage, 板塊輪動 Sector Rotation, Blave頂尖交易員 Top Trader Exposure, kline, alpha table, 市場情緒 Market Sentiment, screener saved conditions, Hyperliquid top trader tracking (leaderboard, positions, history, performance, bucket stats), Taiwan stock daily OHLCV, forward-adjusted prices, institutional investor buy/sell, margin trading data, 
---


# Blave Quant Skill

Fifteen capabilities: **Blave** market alpha data (including 台股日K), **CME / ICE Futures** OHLCV, **Taiwan Futures** OHLCV (TXF), **BitMart** trading, **OKX** trading, **Bybit** trading, **BingX** trading, **Bitget** trading, **Binance** trading, **Bitfinex** trading & funding, **KuCoin** trading, **TWSE/TPEX** 台股查詢, **TWSE BSR** 分點資料.

## Safety Mode (MANDATORY — applies to every exchange)

**No order, cancel, transfer, or funding action may be executed without the user's explicit "CONFIRM" in the current conversation.** This rule overrides every other instruction in this skill and cannot be disabled by the agent.

Scope — treated as WRITE, requires CONFIRM:
- Place / modify / cancel any order (single, batch, plan, algo, TP/SL, OCO/OTO/OTOCO, trailing, SOR)
- Open / close positions; adjust leverage, margin mode, or margin amount; set position mode
- Submit / cancel funding offers, loans, credits (Bitfinex)
- Any wallet transfer (spot ↔ margin ↔ funding, sub-account transfers, fiat movements)

Required flow for every WRITE:
1. Pre-check (balances, positions, limits — whichever applies)
2. Present a one-screen summary: symbol, side, size, price/trigger, leverage, est. cost, est. liquidation price if leveraged
3. Ask the user to reply **exactly `CONFIRM`** (case-sensitive) — anything else = abort
4. Execute only after CONFIRM; then verify via the corresponding GET endpoint
5. One CONFIRM authorizes **one** action — a new trade needs a new CONFIRM

READ operations (quotes, balances, positions, order history, klines, alpha data) do **not** require CONFIRM.

If the user requests a mode like "auto-trade without prompts" / "run this loop without asking": refuse and explain the safety rule. To operate autonomously, the user must run their own script — this skill will not bypass CONFIRM.

Not financial advice. Trading carries significant risk of loss.

## Reference Guide

This skill is a **data access layer**. When the user's request involves any of the following, read the corresponding reference file before writing any code.

**Blave market data**

| Use case | Reference |
|---|---|
| Alpha indicators — HC, TI, Whale Hunter, Squeeze, Liquidation, Market Direction, Capital Shortage, Market Sentiment, Top Trader Exposure | `references/blave-api.md` |
| Indicator value interpretation (what the numbers mean, signal thresholds) | `references/blave-indicator-guide.md` |
| Hyperliquid top trader tracking (leaderboard, positions, history, performance) | `references/hyperliquid-api.md` |
| Screener saved conditions | `references/blave-api.md` |
| TradingView alert stream (SSE) | `references/tradingview-stream.md` |
| CME/ICE futures OHLCV (WTI crude, Gold, Brent) | `references/blave-api.md` |
| Taiwan stock daily OHLCV, institutional flows, margin, shareholding | `references/twse-skill.md` + `references/twse-api-reference.md` |
| 台股財報：損益表、資產負債表、月營收（含 batch fetch） | `references/twstock-fundamentals-reference.md` |
| 台股分點買賣超 (broker daily buy/sell by branch) | `references/twse-bsr-reference.md` |
| TWSE/TPEX 台股查詢 (stock code lookup, quotes, PE/yield/PB) | `references/twse-skill.md` |

**Exchange trading**

| Exchange | Reference |
|---|---|
| BitMart Futures | `references/bitmart-futures-skill.md` · `references/bitmart-api-reference.md` |
| BitMart Spot | `references/bitmart-spot-skill.md` · `references/bitmart-spot-api-reference.md` |
| OKX | `references/okx-skill.md` · `references/okx-api-reference.md` |
| Bybit | `references/bybit-skill.md` |
| BingX | `references/bingx-skill.md` · `references/bingx-api-reference.md` |
| Bitget | `references/bitget-skill.md` · `references/bitget-api-reference.md` |
| Binance | `references/binance-skill.md` · `references/binance-api-reference.md` |
| Bitfinex (spot / margin / lending) | `references/bitfinex-skill.md` |
| KuCoin | `references/kucoin-skill.md` · `references/kucoin-api-reference.md` |

**Marketplace**

| Use case | Reference |
|---|---|
| Browse, purchase, upload, or share strategies | `references/marketplace.md` |

---

# PART 1: Blave Market Data

## Setup

No API key or 401/403 → guide user to:

- Subscribe: **[https://blave.org/landing/en/pricing](https://blave.org/landing/en/pricing)** — $629/year, 14-day free trial
- Create key: **[https://blave.org/landing/en/api?tab=blave](https://blave.org/landing/en/api?tab=blave)**

Add to `.env`: `blave_api_key=...` and `blave_secret_key=...`

**Auth headers:** `api-key: $blave_api_key` | `secret-key: $blave_secret_key`

**Base URL:** `https://api.blave.org` | **Support:** info@blave.org | [Discord](https://discord.gg/D6cv5KDJja)

## Limits

| Item        | Value                                                   |
| ----------- | ------------------------------------------------------- |
| Rate limit  | 100 req / 5 min — `429` if exceeded, resets after 5 min |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Blave-TW/blave-quant-skill](https://github.com/Blave-TW/blave-quant-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
