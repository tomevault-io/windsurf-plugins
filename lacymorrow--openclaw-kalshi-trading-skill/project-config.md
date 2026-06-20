---
trigger: always_on
description: Trade prediction markets on Kalshi using the kalshi-cli command-line tool. Use when the user wants to trade event contracts, browse prediction markets, place orders, manage positions, stream live prices, or view candlestick charts. Also use when they mention "prediction market," "event contract," "kalshi," "YES/NO," "order book," "limit order," "market order," "hedge," "arbitrage," or "liquidity rewards.
---


# Kalshi Trading Skill

Trade CFTC-regulated prediction markets on Kalshi through the `kalshi-cli` command-line tool. Browse markets, place limit and market orders, manage positions, stream real-time prices, and view ASCII candlestick charts — all from the terminal.

Kalshi is fully legal in the US. No crypto wallets, no blockchain, no gas fees — everything settles in USD.

## Installation

### macOS / Linux (Homebrew)
```bash
brew install 6missedcalls/tap/kalshi-cli
```

### Go Install (requires Go 1.25+)
```bash
go install github.com/6missedcalls/kalshi-cli/cmd/kalshi-cli@latest
```

### Build from Source
```bash
git clone https://github.com/6missedcalls/kalshi-cli.git
cd kalshi-cli
go build -o kalshi-cli ./cmd/kalshi-cli
```

## Authentication

### Interactive Login (Recommended first time)
```bash
kalshi-cli auth login
```

Follow the prompts to:
1. Copy the displayed public key
2. Add it to your Kalshi account at https://kalshi.com/account/api-keys
3. Enter the API Key ID when prompted

Credentials are stored securely in your OS keyring (macOS Keychain, GNOME Keyring, or Windows Credential Manager).

### Non-Interactive Login (Bots / CI)
```bash
# Via flags
kalshi-cli auth login --api-key-id YOUR_KEY_ID --private-key-file /path/to/key.pem

# Via PEM content
kalshi-cli auth login --api-key-id YOUR_KEY_ID --private-key "$(cat /path/to/key.pem)"

# Via environment variables
export KALSHI_API_KEY_ID=your-key-id
export KALSHI_PRIVATE_KEY="$(cat /path/to/key.pem)"
kalshi-cli auth login
```

### Config File
Add to `~/.kalshi/config.yaml`:
```yaml
api_key_id: your-key-id
private_key_path: /path/to/key.pem
```

Credentials are resolved in order: config file, environment variables, OS keyring.

### Demo vs Production

| | Demo (default) | Production |
|--|------|-----------|
| Flag | (default) | `--prod` |
| API | `demo-api.kalshi.co` | `api.elections.kalshi.com` |
| Money | Fake/test | Real USD |

**The CLI defaults to demo mode.** You must pass `--prod` to trade with real money. This is a safety feature.

## Overview

You are an expert in using `kalshi-cli` for prediction market trading on Kalshi. Your goal is to help users trade event contracts efficiently while emphasizing safety and risk awareness.

## How to Use This Skill

1. **Safety First**: The CLI defaults to demo mode. Only use `--prod` when the user explicitly wants real trading.
2. **Verify Before Trading**: Always show the exact command and confirm with the user before executing trades.
3. **Check Prerequisites**: Confirm auth is set up (`auth status`), exchange is active (`exchange status`), and sufficient balance exists.
4. **Use JSON Output**: For scripting and automation, use `--json` for machine-readable output.
5. **Understand Tickers**: Markets use structured tickers like `KXBTC-26FEB12-B97000`. Events group related markets.

## When to Use This Skill

Use this skill when the user wants to:
- Browse or search prediction markets (politics, crypto, sports, economics, weather)
- Place bets (limit orders, market orders) on event contracts
- Check market prices, spreads, and order books
- View ASCII candlestick charts
- Stream live price updates via WebSocket
- View or manage open orders
- Check portfolio positions and P/L
- Cancel or amend orders
- Monitor trade fills and settlements
- Use batch orders for market-making strategies

**Common trigger phrases:**
- "What prediction markets are open on Kalshi?"
- "Buy YES on Bitcoin above 100k"
- "Show me the order book"
- "Stream live prices for this market"
- "Show me a candlestick chart"
- "Cancel all my orders"
- "Check my positions"
- "Place a limit order at 50 cents"
- "What's my balance?"

**When NOT to use this skill:**
- User wants to trade stocks/ETFs/options (use Alpaca trading skill)
- User wants to trade on Polymarket (use Polymarket trading skill)
- User wants financial advice (provide tools, not recommendations)

## Key Concepts

### Tickers
Markets use structured tickers: `KXBTC-26FEB12-B97000`
- `KXBTC` = series (Bitcoin)
- `26FEB12` = date (Feb 12, 2026)
- `B97000` = strike ($97,000)

### Events vs Markets
An **event** groups related markets. For example, "Bitcoin price on Feb 12" is an event containing markets at different strike prices ($90K, $95K, $100K, etc.).

### Contracts
Kalshi trades **event contracts** priced 1-99 cents, settling to $1 or $0. All values are in **cents**.

### Subaccounts
Up to 32 subaccounts per user for multi-strategy isolation.

### Order Groups
Cap total fills across multiple orders — useful for market-making where you don't want both sides filled.

## Global Flags

Every command accepts these flags:

| Flag | Short | Default | Description |
|------|-------|---------|-------------|
| `--json` | | `false` | Output as JSON for scripts |
| `--plain` | | `false` | Plain text for piping |
| `--yes` | `-y` | `false` | Skip all confirmation prompts |
| `--prod` | | `false` | Use production API (real money) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lacymorrow/openclaw-kalshi-trading-skill](https://github.com/lacymorrow/openclaw-kalshi-trading-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
