---
trigger: always_on
description: Fintool is a suite of Rust CLI tools for agentic trading and market intelligence across multiple exchanges. Each exchange has its own dedicated binary. All CLIs support `--json` mode for scripting and AI agent integration.
---

# CLAUDE.md — Fintool Development Guide

## Project Overview

Fintool is a suite of Rust CLI tools for agentic trading and market intelligence across multiple exchanges. Each exchange has its own dedicated binary. All CLIs support `--json` mode for scripting and AI agent integration.

**Supported exchanges**: Hyperliquid, Binance, Coinbase, OKX, Polymarket (prediction markets)
**Asset classes**: Crypto, stocks, commodities, indices, prediction markets
**License**: MIT

## Repository Structure

```
fintool/
├── src/
│   ├── lib.rs              # Library root — exports all modules
│   ├── bin/                 # Binary entry points (one per exchange + fintool + backtest)
│   │   ├── fintool.rs       # Market intelligence CLI (quotes, news, SEC filings)
│   │   ├── hyperliquid.rs   # Hyperliquid (spot, perp, HIP-3, deposits, withdrawals)
│   │   ├── binance.rs       # Binance (spot, perp, deposits, withdrawals)
│   │   ├── coinbase.rs      # Coinbase (spot, deposits, withdrawals)
│   │   ├── okx.rs           # OKX (spot, perp, deposits, withdrawals, transfers)
│   │   ├── polymarket.rs    # Polymarket (prediction market trading)
│   │   └── backtest.rs      # Historical simulation with forward PnL analysis
│   ├── commands/            # Shared command implementations used by binaries
│   │   ├── quote.rs         # Multi-source price quotes + LLM enrichment
│   │   ├── news.rs          # News via Google News RSS
│   │   ├── report.rs        # SEC EDGAR filings
│   │   ├── order.rs         # Spot order placement
│   │   ├── perp.rs          # Perpetual futures trading
│   │   ├── deposit.rs       # Deposit flows (bridging, address generation)
│   │   ├── withdraw.rs      # Withdrawal flows
│   │   ├── balance.rs       # Account balances
│   │   ├── positions.rs     # Open positions
│   │   ├── orders.rs        # Order listing
│   │   ├── cancel.rs        # Order cancellation
│   │   ├── orderbook.rs     # Orderbook depth
│   │   ├── transfer.rs      # Internal account transfers
│   │   ├── options.rs       # Options trading
│   │   ├── predict.rs       # Polymarket prediction operations
│   │   └── bridge_status.rs # Cross-chain bridge status
│   ├── config.rs            # Config loading (~/.fintool/config.toml)
│   ├── signing.rs           # Hyperliquid EIP-712 wallet signing
│   ├── hip3.rs              # HIP-3 builder-deployed perps (collateral tokens)
│   ├── backtest.rs          # Historical data providers + simulated portfolio
│   ├── binance.rs           # Binance REST API client
│   ├── coinbase.rs          # Coinbase API client
│   ├── okx.rs               # OKX API client
│   ├── polymarket.rs        # Polymarket SDK helpers
│   ├── bridge.rs            # Across Protocol cross-chain bridging
│   ├── unit.rs              # HyperUnit bridge (ETH/BTC/SOL)
│   └── format.rs            # Color formatting + number formatting helpers
├── tests/                   # E2E shell script tests organized by exchange
│   ├── helpers.sh           # Shared test utilities (build checks, assertions)
│   ├── backtest/            # Backtest CLI tests
│   ├── hyperliquid/         # Hyperliquid E2E tests
│   ├── binance/             # Binance E2E tests
│   ├── okx/                 # OKX E2E tests
│   └── polymarket/          # Polymarket E2E tests
├── examples/                # Complete example scripts (see Examples section below)
│   ├── funding_arb/         # Funding rate arbitrage bot
│   ├── metal_pair/          # Metal pairs trading bot
│   ├── catalyst_hedger/     # Prediction market hedging bot
│   └── backtest/            # Historical backtest strategy examples
├── skills/                  # AI agent skill definitions (for OpenClaw / Claude)
│   ├── SKILL.md             # Main skill definition (commands, workflows, capabilities)
│   ├── install.md           # Installation guide for agents
│   └── bootstrap.sh         # Automated installer script
├── docs/                    # Additional documentation and screenshots
├── .github/workflows/       # CI/CD (ci.yml for lint+test, release.yml for builds)
├── Cargo.toml               # Rust dependencies and binary targets
├── config.toml.default      # Default config template
└── README.md                # User-facing documentation
```

## Building

```bash
# Debug build
cargo build

# Release build (used for testing and deployment)
cargo build --release
```

Binaries are output to `target/release/` (or `target/debug/`):
`fintool`, `hyperliquid`, `binance`, `coinbase`, `okx`, `polymarket`, `backtest`

## Testing

### Lint (must pass before submitting PRs)

```bash
# Format check — CI runs this exact command
cargo fmt -- --check

# Auto-fix formatting
cargo fmt

# Clippy — CI runs with warnings as errors
cargo clippy --release -- -D warnings
```

### Unit tests

```bash
cargo test --release
```

### E2E tests

Shell script tests in `tests/` organized by exchange. Each exchange directory has an `e2e_*.sh` script that runs the full workflow:

```bash
# Run backtest E2E tests (no API keys needed)
bash tests/backtest/e2e_backtest.sh

# Run individual tests
bash tests/backtest/quote_btc.sh

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [second-state/fintool](https://github.com/second-state/fintool) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
