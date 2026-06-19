---
trigger: always_on
description: WEEX Futures exchange integration. Trade USDT-M perpetual futures with up to 125x leverage on WEEX.
---


# WEEX Futures Trading 🔵

Open AI Agent Skill for USDT-margined perpetual futures trading on WEEX exchange. Up to 125x leverage.

> **Open Agent Skill**: This skill is designed to work with any AI agent that supports bash/curl commands, including Claude, GPT, Gemini, LLaMA, Mistral, and other LLM-based agents.

## Features

- 📊 **Futures Trading** - USDT-M perpetual contracts up to 125x leverage
- 💰 **Account Management** - Balance, positions, margin settings
- 📈 **Market Data** - Tickers, order book, candlesticks, funding rates
- 🎯 **Advanced Orders** - Trigger orders, TP/SL, conditional orders
- 🤖 **AI Integration** - Log AI trading decisions
- 🔌 **Universal Compatibility** - Works with any AI agent supporting shell commands

## Environment Variables

| Variable | Description | Required |
|----------|-------------|----------|
| `WEEX_API_KEY` | API Key from WEEX | Yes |
| `WEEX_API_SECRET` | API Secret | Yes |
| `WEEX_PASSPHRASE` | API Passphrase | Yes |
| `WEEX_BASE_URL` | API base URL | No (default: https://api-contract.weex.com) |

## Authentication

```bash
API_KEY="${WEEX_API_KEY}"
SECRET="${WEEX_API_SECRET}"
PASSPHRASE="${WEEX_PASSPHRASE}"
BASE_URL="${WEEX_BASE_URL:-https://api-contract.weex.com}"

TIMESTAMP=$(python3 -c "import time; print(int(time.time() * 1000))")

# Generate signature
generate_signature() {
  local method="$1"
  local path="$2"
  local body="$3"
  local message="${TIMESTAMP}${method}${path}${body}"
  echo -n "$message" | openssl dgst -sha256 -hmac "$SECRET" -binary | base64
}
```

---

# Market Data Endpoints (No Auth)

## Get Server Time

```bash
curl -s "${BASE_URL}/capi/v2/market/time" | jq '.'
```

## Get All Contracts Info

```bash
curl -s "${BASE_URL}/capi/v2/market/contracts" | jq '.data[] | {symbol: .symbol, baseCoin: .underlying_index, quoteCoin: .quote_currency, contractVal: .contract_val, minLeverage: .minLeverage, maxLeverage: .maxLeverage, tickSize: .tick_size, sizeIncrement: .size_increment}'
```

## Get Single Contract Info

```bash
SYMBOL="cmt_btcusdt"

curl -s "${BASE_URL}/capi/v2/market/contracts?symbol=${SYMBOL}" | jq '.data'
```

## Get Ticker Price

```bash
SYMBOL="cmt_btcusdt"

curl -s "${BASE_URL}/capi/v2/market/ticker?symbol=${SYMBOL}" | jq '.data | {symbol: .symbol, last: .last, high: .high_24h, low: .low_24h, volume: .volume_24h, markPrice: .markPrice}'
```

## Get All Tickers

```bash
curl -s "${BASE_URL}/capi/v2/market/tickers" | jq '.data[] | {symbol: .symbol, last: .last, change: .priceChangePercent, volume: .volume_24h}'
```

## Get Order Book

```bash
SYMBOL="cmt_btcusdt"

curl -s "${BASE_URL}/capi/v2/market/depth?symbol=${SYMBOL}&limit=15" | jq '.data | {asks: .asks[:5], bids: .bids[:5]}'
```

## Get Recent Trades

```bash
SYMBOL="cmt_btcusdt"
LIMIT="50"

curl -s "${BASE_URL}/capi/v2/market/trades?symbol=${SYMBOL}&limit=${LIMIT}" | jq '.data[] | {time: .time, price: .price, size: .size, side: (if .isBuyerMaker then "sell" else "buy" end)}'
```

## Get Candlestick Data

```bash
SYMBOL="cmt_btcusdt"
GRANULARITY="1h"    # 1m, 5m, 15m, 30m, 1h, 4h, 12h, 1d, 1w
LIMIT="100"

curl -s "${BASE_URL}/capi/v2/market/candles?symbol=${SYMBOL}&granularity=${GRANULARITY}&limit=${LIMIT}" | jq '.data[] | {timestamp: .[0], open: .[1], high: .[2], low: .[3], close: .[4], volume: .[5]}'
```

## Get Index Price

```bash
SYMBOL="cmt_btcusdt"

curl -s "${BASE_URL}/capi/v2/market/index?symbol=${SYMBOL}" | jq '.data | {symbol: .symbol, index: .index, timestamp: .timestamp}'
```

## Get Open Interest

```bash
SYMBOL="cmt_btcusdt"

curl -s "${BASE_URL}/capi/v2/market/open_interest?symbol=${SYMBOL}" | jq '.data[] | {symbol: .symbol, openInterest: .base_volume, value: .target_volume}'
```

## Get Current Funding Rate

```bash
SYMBOL="cmt_btcusdt"

curl -s "${BASE_URL}/capi/v2/market/currentFundRate?symbol=${SYMBOL}" | jq '.data[] | {symbol: .symbol, rate: .fundingRate, nextSettlement: .timestamp}'
```

## Get Historical Funding Rates

```bash
SYMBOL="cmt_btcusdt"
LIMIT="20"

curl -s "${BASE_URL}/capi/v2/market/getHistoryFundRate?symbol=${SYMBOL}&limit=${LIMIT}" | jq '.data[] | {symbol: .symbol, rate: .fundingRate, settleTime: .fundingTime}'
```

## Get Next Funding Time

```bash
SYMBOL="cmt_btcusdt"

curl -s "${BASE_URL}/capi/v2/market/funding_time?symbol=${SYMBOL}" | jq '.data | {symbol: .symbol, nextFundingTime: .fundingTime}'
```

---

# Account Endpoints (Auth Required)

## Get Account Assets

```bash
PATH_URL="/capi/v2/account/assets"
TIMESTAMP=$(python3 -c "import time; print(int(time.time() * 1000))")
SIGNATURE=$(generate_signature "GET" "$PATH_URL" "")

curl -s "${BASE_URL}${PATH_URL}" \
  -H "ACCESS-KEY: ${API_KEY}" \
  -H "ACCESS-SIGN: ${SIGNATURE}" \
  -H "ACCESS-PASSPHRASE: ${PASSPHRASE}" \
  -H "ACCESS-TIMESTAMP: ${TIMESTAMP}" \
  -H "Content-Type: application/json" | jq '.data[] | {coin: .coinName, available: .available, frozen: .frozen, equity: .equity, unrealizedPnl: .unrealizePnl}'
```

## Get Account List with Settings

```bash
PATH_URL="/capi/v2/account/getAccounts"
TIMESTAMP=$(python3 -c "import time; print(int(time.time() * 1000))")
SIGNATURE=$(generate_signature "GET" "$PATH_URL" "")

curl -s "${BASE_URL}${PATH_URL}" \
  -H "ACCESS-KEY: ${API_KEY}" \
  -H "ACCESS-SIGN: ${SIGNATURE}" \

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bowen31337/weex-futures-trading-skill](https://github.com/bowen31337/weex-futures-trading-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
