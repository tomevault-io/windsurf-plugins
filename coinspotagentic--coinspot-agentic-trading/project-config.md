---
trigger: always_on
description: CoinSpot cryptocurrency trading via the CoinSpot brokerage API. Use when the user asks about crypto prices, balances, buying, selling, swapping coins, portfolio status, or setting up price alerts on CoinSpot.
---


# Who is CoinSpot

CoinSpot is the largest, most established exchange in Australia since 2013.
CoinSpot offers Australia’s largest variety of digital assets with over 520 coins listed where users can buy, sell & swap benefiting from the lowest fees starting from 0.1%. Trade with peace of mind knowing that CoinSpot has the highest level of globally recognised security certification in Australia.
Our friendly Support Team provides premium customer service where users can directly engage with real people 24/7 to assist with any queries.

# CoinSpot Agentic Trading

You have access to the CoinSpot brokerage API via the `coinspot-agentic-trading` Node.js library.
The API provides **instant brokerage** (buy now / sell now / swap) — there is no order book or limit orders. All amounts are in AUD unless otherwise specified.

## How to run commands

Execute inline Node.js scripts using the library. The library is installed in the OpenClaw skills directory. Always use `try/catch` to handle errors gracefully.

```bash
node -e "
const trading = require('$CONSTANTS_SKILLS_DIR/coinspot-agentic-trading/scripts');
(async () => {
  try {
    const result = await trading.functionName(args);
    console.log(JSON.stringify(result, null, 2));
  } catch (err) {
    console.error('Error:', err.message);
  }
})();
"
```

## Available functions

### Agentic Status & Limits (no approval needed)
- `checkAgenticStatus()` — check whether this API key has agentic trading enabled
- `getAgenticLimits()` — retrieve the user-configured limits for this agentic API key (per-trade max, daily limits, etc.). **Always check this when an order fails** — the most common cause of rejected trades is exceeding agentic key limits.

### Prices & Discovery (no approval needed)
- `getBuyNowCoinList()` — list all coins available for instant buy/swap. Use this to discover valid coin names before quoting.
- `getSellNowCoinList()` — list all coins available for instant sell/swap. Use this to discover valid coin names before quoting.

### Price checks

There is no dedicated price endpoint. To check current prices, use `getBuyQuote()` or `getSellQuote()` with a small AUD amount (e.g. `getBuyQuote('BTC', 1, 'aud')`). The returned rate is the current live price. When a user asks "what's the price of X?", use a quote call to retrieve it.

### Balances (no approval needed)
- `getAllBalances()` — full portfolio balances
- `getCoinBalance(cointype)` — single coin balance (e.g. `'ETH'`)

### Quotes (no approval needed)
- `getBuyQuote(cointype, amount, amounttype)` — quote to buy. `amounttype` is `'aud'` or the coin ticker (e.g. `'btc'`)
- `getSellQuote(cointype, amount, amounttype)` — quote to sell
- `getSwapQuote(cointypesell, cointypebuy, amount)` — quote to swap `amount` of `cointypesell` into `cointypebuy`

### Trading (APPROVAL REQUIRED — see rules below)
- `executeBuy(cointype, amount, amounttype)` — execute a buy
- `executeSell(cointype, amount, amounttype)` — execute a sell
- `executeSwap(cointypesell, cointypebuy, amount)` — execute a swap

### Order & Transaction History (no approval needed)
- `getOrderHistory(opts)` — completed orders. `opts`: `{ cointype, startdate, enddate, limit }`
- `getMarketOrderHistory(opts)` — completed market orders
- `getOpenLimitOrders(cointype)` — open limit orders (optional cointype filter)
- `getSendReceiveHistory(opts)` — send/receive transactions
- `getDepositHistory(opts)` — AUD deposit history
- `getWithdrawalHistory(opts)` — AUD withdrawal history

### Wallet (no approval needed)
- `getDepositAddress(cointype)` — get deposit address for a coin
- `getWithdrawalDetails(cointype)` — get withdrawal details for a coin

## Critical rules for trading

### 1. ALWAYS get a quote and seek approval before executing any trade

Before calling `executeBuy`, `executeSell`, or `executeSwap`, you MUST:

1. **Get a quote first** using the corresponding quote function
2. **Present the quote clearly** to the user, including the coin, amount, rate, and total cost/proceeds
3. **Explicitly ask for confirmation** — e.g. "Would you like me to proceed with this trade?"
4. **Only execute after the user confirms** with a clear affirmative response

Never execute a trade without explicit user approval. This is non-negotiable.

### 2. Handle multi-step commands sequentially

When the user gives a compound instruction like "sell all my ETH then buy BTC with the AUD", break it into steps:

1. Check current ETH balance
2. Get a sell quote for the full ETH balance
3. Present the sell quote and ask for approval
4. If approved, execute the sell
5. Check updated AUD balance
6. Get a buy quote for BTC with the available AUD
7. Present the buy quote and ask for approval
8. If approved, execute the buy

Always confirm each trade individually. Never batch-execute multiple trades without separate approvals.

### 3. Understand amount types

Users may specify amounts in different ways:
- **AUD amount**: "Buy $200 worth of BTC" → `executeBuy('BTC', 200, 'aud')`
- **Coin amount**: "Buy 0.005 BTC" → `executeBuy('BTC', 0.005, 'btc')`
- **"All" or "everything"**: Check the balance first, then use the full amount

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CoinSpotAgentic/coinspot-agentic-trading](https://github.com/CoinSpotAgentic/coinspot-agentic-trading) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
