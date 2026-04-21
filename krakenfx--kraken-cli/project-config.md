---
trigger: always_on
description: > **This is experimental software. Commands execute real financial transactions on the Kraken exchange. Test with `kraken paper` before using real funds. See [DISCLAIMER.md](DISCLAIMER.md) for full terms.**
---

# Agent Integration Guide: kraken-cli

> **This is experimental software. Commands execute real financial transactions on the Kraken exchange. Test with `kraken paper` before using real funds. See [DISCLAIMER.md](DISCLAIMER.md) for full terms.**

Self-contained guide for integrating `kraken-cli` into AI agents, MCP clients, and automated pipelines.

Fast entry points:
- Runtime agent context: `CONTEXT.md`
- Full command contract: `agents/tool-catalog.json`
- Error routing contract: `agents/error-catalog.json`
- Workflow skills: `skills/`

## Installation

Single binary, no runtime dependencies.

```bash
curl --proto '=https' --tlsv1.2 -LsSf https://github.com/krakenfx/kraken-cli/releases/latest/download/kraken-cli-installer.sh | sh
```

Pre-built binaries for macOS (Apple Silicon, Intel) and Linux (x86_64, ARM64) are also available on the [GitHub Releases](https://github.com/krakenfx/kraken-cli/releases) page.

Verify: `kraken --version`

## Authentication

### Environment variables (recommended for agents)

```bash
export KRAKEN_API_KEY="your-key"
export KRAKEN_API_SECRET="your-secret"

# Futures (optional, separate credentials)
export KRAKEN_FUTURES_API_KEY="your-futures-key"
export KRAKEN_FUTURES_API_SECRET="your-futures-secret"
```

No config file needed. No interactive prompts. The CLI reads credentials from the environment.

### Credential resolution order

1. Command-line flags (`--api-key`, `--api-secret`)
2. Environment variables (`KRAKEN_API_KEY`, `KRAKEN_API_SECRET`)
3. Config file (`~/.config/kraken/config.toml`)

### Required permissions by command group

| Group | Kraken API permissions |
|-------|----------------------|
| market | None (public) |
| account | Query Funds, Query Open Orders & Trades, Query Closed Orders & Trades, Query Ledger Entries |
| trade | Create & Modify Orders, Cancel/Close Orders |
| funding | Deposit Funds, Withdraw Funds, Query Funds |
| earn | Query Funds, Create & Modify Orders |
| subaccount | Access & Modify Account Settings |
| websocket (public) | None |
| websocket (private) | Same as account + trade |
| futures | Access Futures Markets, Create & Modify Orders, Cancel/Close Orders |
| paper | None (local simulation) |
| futures paper | None (local simulation) |

## Invocation Pattern

Every command follows the same pattern:

```bash
kraken <command> [args...] -o json 2>/dev/null
```

Rules:
- Always pass `-o json` for structured output.
- Redirect stderr: `2>/dev/null` or `2>debug.log`.
- stdout contains only valid JSON (success) or a JSON error envelope (failure).
- Exit code 0 = success, non-zero = failure.

### Basic examples

```bash
# Public data (no auth)
kraken ticker BTCUSD -o json
kraken orderbook BTCUSD --count 10 -o json
kraken ohlc BTCUSD --interval 60 -o json

# Private data
kraken balance -o json
kraken open-orders -o json

# Trading
kraken order buy BTCUSD 0.001 --type limit --price 50000 -o json
kraken order cancel TXID123 -o json

# Spot paper trading (no auth)
kraken paper init --balance 10000 -o json
kraken paper buy BTCUSD 0.01 -o json
kraken paper status -o json

# Futures paper trading (no auth)
kraken futures paper init --balance 10000 -o json
kraken futures paper buy PF_XBTUSD 1 --leverage 10 --type market -o json
kraken futures paper positions -o json
```

## Asset Classes

The CLI supports six asset classes through the same command surface. Most commands default to crypto. For other asset classes, pass the `--asset-class` flag.

### Crypto spot (default)

No flag needed. Covers 1,400+ pairs with margin up to 10x on major pairs.

```bash
kraken ticker BTCUSD -o json
kraken order buy BTCUSD 0.001 --type limit --price 50000 -o json
kraken order buy BTCUSD 0.001 --type limit --price 50000 --leverage 5 -o json
```

### Tokenized U.S. stocks & ETFs (xStocks)

Requires `--asset-class tokenized_asset`. Stock symbols use the `x` suffix: AAPL becomes `AAPLx`, TSLA becomes `TSLAx`. 79 assets available with margin up to 3x on the top 10. Not available in the USA.

```bash
kraken ticker AAPLx/USD --asset-class tokenized_asset -o json
kraken order buy AAPLx/USD 0.1 --type limit --price 200 --asset-class tokenized_asset -o json
kraken order buy TSLAx/USD 0.5 --type limit --price 250 --asset-class tokenized_asset --leverage 3 -o json
```

To list all xStocks assets: `kraken assets --asset-class tokenized_asset -o json`

### Forex

Requires `--asset-class forex` on market data commands. 11 fiat pairs including EUR/USD, GBP/USD, USD/JPY, AUD/USD.

```bash
kraken ticker EURUSD --asset-class forex -o json
```

### Futures

Separate engine with its own credentials (`KRAKEN_FUTURES_API_KEY`, `KRAKEN_FUTURES_API_SECRET`). Covers crypto perpetuals, 5 forex perps (PF_EURUSD, PF_GBPUSD, PF_AUDUSD, PF_CHFUSD, PF_JPYUSD), 11 equity/index perps (PF_AAPLXUSD, PF_NVDAXUSD, PF_TSLAXUSD, PF_SPYXUSD, PF_QQQXUSD, PF_SPXUSD, and others), and fixed-date contracts. Leverage up to 50x.

```bash
kraken futures instruments -o json                                            # list all contracts
kraken futures order buy PF_XBTUSD 1 --type limit --price 50000 -o json      # crypto perp

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [krakenfx/kraken-cli](https://github.com/krakenfx/kraken-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
