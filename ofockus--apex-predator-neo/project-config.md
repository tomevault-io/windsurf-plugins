---
trigger: always_on
description: Autonomous triangular arbitrage scanner and AI trading assistant for Binance. Combines all 7 official Binance AI Agent Skills into a unified HFT-grade workflow with real-time triangle detection, spoof hunting, risk management, and natural language control. Built for the Binance OpenClaw Challenge.
---


# APEX PREDATOR NEO v666 🦈

## Autonomous Triangular Arbitrage Scanner + AI Trading Assistant for Binance

**An OpenClaw skill that orchestrates ALL 7 official Binance AI Agent Skills into one unified, autonomous HFT-grade trading workflow.**

> "7 AI modules. Zero human intervention. Sub-40ms scan cycles."

---

## What It Does

APEX PREDATOR NEO transforms your OpenClaw agent into a professional-grade crypto trading assistant that:

1. **Scans triangular arbitrage opportunities** across 800+ Binance Spot pairs in real-time
2. **Filters signals** through a 6-layer ConfluenceEngine (fake momentum rejection, spoof detection, OI consistency)
3. **Manages risk autonomously** with hard drawdown limits and automatic position freezing
4. **Sweeps profits** into Binance Simple Earn automatically
5. **Reports everything** via your preferred messaging channel (WhatsApp, Telegram, Slack, Discord)

---

## Environment Variables

| Variable | Description | Required |
|----------|-------------|----------|
| `BINANCE_API_KEY` | API Key from Binance (Spot trading enabled) | Yes |
| `BINANCE_SECRET` | API Secret | Yes |
| `APEX_MODE` | `testnet` (default) or `live` | No |
| `APEX_CAPITAL` | Total capital in USDT (default: 22) | No |
| `APEX_MAX_TRADE` | Max per trade in USDT (default: 8) | No |
| `APEX_MAX_DRAWDOWN` | Max drawdown % before freeze (default: 4.0) | No |
| `APEX_SCAN_INTERVAL_MS` | Scan cycle interval (default: 40) | No |
| `APEX_AUTO_EARN` | Enable auto-earn sweep (default: true) | No |
| `APEX_EARN_THRESHOLD` | Min profit to sweep to Earn (default: 0.10) | No |

---

## Binance Skills Integration

This skill orchestrates all 7 official Binance AI Agent Skills:

### 1. CEX Spot Trading Skill
**Used for:** Real-time market data (tickers, depth, candlesticks) and trade execution (market/limit/OCO orders).

```bash
# Fetch current BTC price
curl -s "https://api.binance.com/api/v3/ticker/price?symbol=BTCUSDT" | jq '.price'

# Fetch orderbook depth (15 levels)
curl -s "https://api.binance.com/api/v3/depth?symbol=BTCUSDT&limit=15" | jq '.'

# Fetch all tickers for triangle scanning
curl -s "https://api.binance.com/api/v3/ticker/bookTicker" | jq '.'

# Place testnet order (HMAC-SHA256 signed)
TIMESTAMP=$(date +%s%3N)
QUERY="symbol=BTCUSDT&side=BUY&type=MARKET&quoteOrderQty=8&timestamp=${TIMESTAMP}"
SIGNATURE=$(echo -n "${QUERY}" | openssl dgst -sha256 -hmac "${BINANCE_SECRET}" | cut -d' ' -f2)
curl -s -X POST "https://testnet.binance.vision/api/v3/order?${QUERY}&signature=${SIGNATURE}" \
  -H "X-MBX-APIKEY: ${BINANCE_API_KEY}" | jq '.'
```

### 2. Address Insight Skill
**Used for:** Whale tracking — monitor large wallet movements that could affect triangle spreads.

```bash
# Analyze a whale wallet for holdings and concentration
# Used by EconoPredator module to detect institutional flow
```

### 3. Token Details Skill
**Used for:** Validate token liquidity and trading pairs before including in triangle paths.

### 4. Market Rankings Skill
**Used for:** Dynamic pair selection — auto-select top pairs by volume/volatility for optimal triangle discovery.

### 5. Meme Rush Skill
**Used for:** Detect sudden meme coin surges that create temporary arbitrage dislocations across pairs.

### 6. Trading Signals Skill
**Used for:** Confluence layer — cross-reference APEX signals with Binance's own signal engine for higher confidence.

### 7. Token Contract Audit Skill
**Used for:** Anti-rug module — audit token contracts before including new pairs in triangle scan paths.

---

## 7 AI Modules

### 🧠 ConfluenceEngine — 6-Layer Signal Filter
Before any trade executes, the signal must pass ALL 6 filters:

1. **Tire Pressure** — Measures bid-ask spread compression as a proxy for execution certainty
2. **Lead-Lag Detection** — Identifies which pair in the triangle moves first (information advantage)
3. **Fake Momentum Filter** — Rejects signals driven by wash trading or artificial volume
4. **OI Consistency** — Cross-references Open Interest data to confirm real market conviction
5. **OI Delta/Volume Ratio** — Ensures volume is organic and not inflated by derivatives hedging
6. **Post-Spike Reversal** — Avoids entering after sharp moves that are likely to mean-revert

### 🛡️ Robin Hood Risk Engine
Zero-tolerance capital protection:
- **Hard drawdown limit:** 4% max → triggers 30-minute total freeze
- **Per-trade limit:** $8 max (configurable via `APEX_MAX_TRADE`)
- **No override possible** — survival logic is non-negotiable
- **Auto-recovery:** Resumes scanning after cooldown period

### 👁️ SpoofHunter — L2 Defense
Real-time Level 2 orderbook analysis:
- Detects ghost orders (large orders placed then cancelled within 200ms)
- Identifies fake walls and layering patterns
- Flags wash trading activity
- **Any signal contaminated by spoof activity is automatically rejected**

### 📊 EconoPredator — Macro Intelligence
Monitors external factors that affect crypto volatility:
- CPI/FOMC/NFP economic calendar events
- Funding rates across major pairs
- Open Interest shifts

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ofockus/apex-predator-neo](https://github.com/ofockus/apex-predator-neo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
