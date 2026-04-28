---
trigger: always_on
description: This file provides guidance to AI agents when working with code in this repository.
---

# AGENTS.md

This file provides guidance to AI agents when working with code in this repository.

## Development Workflow

Before implementing anything, send me your plan of action for approval.
Start implementing things only after my explicit approval.

Follow test driven development.

## Package Management

Use UV package manager for all commands in this repo:
- `uv sync` - Install/sync dependencies
- `uv add <package>` - Add new dependencies  
- `uv run <command>` - Run commands in the virtual environment
- `uv run <script>` - Run Python scripts

## Project Overview

This is a professional-grade automated grid trading system for Hyperliquid DEX supporting:
- **Spot trading** - Direct asset ownership (cash trading)
- **Perpetuals (perps)** - Leveraged derivatives trading  
- **Grid trading strategies** - Automated buy/sell orders across price ranges
- **Risk management** - Stop loss, take profit, drawdown limits, and position sizing
- **Real-time market data** - WebSocket price feeds and order book data

The codebase follows SOLID principles without overcomplicating the implementation.

## Repository Structure

```
├── bots/                          # Bot configurations (YAML files)
│   └── btc_conservative.yaml      # Conservative BTC grid strategy
├── src/                           # Source code
│   ├── run_bot.py                # Main bot runner (auto-discovers config)
│   ├── core/                     # Core engine components
│   │   ├── engine.py             # Main trading engine
│   │   ├── enhanced_config.py    # Configuration management
│   │   ├── key_manager.py        # Private key management
│   │   ├── risk_manager.py       # Risk management and exit strategies
│   │   └── endpoint_router.py    # API endpoint routing
│   ├── strategies/               # Trading strategies
│   │   └── grid/
│   │       └── basic_grid.py     # Grid trading implementation
│   ├── exchanges/                # Exchange adapters
│   │   └── hyperliquid/
│   │       ├── adapter.py        # Hyperliquid exchange integration
│   │       └── market_data.py    # Real-time market data provider
│   ├── interfaces/               # Business logic interfaces
│   │   ├── strategy.py           # Trading strategy interface
│   │   └── exchange.py           # Exchange adapter interface
│   └── utils/                    # Shared utilities
│       ├── events.py             # Event definitions
│       └── exceptions.py         # Custom exception classes
├── learning_examples/            # Standalone educational scripts
│   ├── 01_websockets/            # Real-time data streams via WebSocket
│   ├── 02_market_data/           # Price data and market info
│   ├── 03_account_info/          # Account state and orders
│   ├── 04_trading/               # Order placement and cancellation
│   ├── 05_funding/               # Funding rates and spot/perp availability
│   └── 06_copy_trading/          # Order mirroring and copy trading strategies
└── .env                          # Environment variables (not in git)
```

## Configuration System

**Bot configurations are stored as YAML files in the `bots/` directory.**

Each configuration includes:
- `active: true/false` - Controls whether bot runs automatically
- `name` - Unique bot identifier
- `account` - Account allocation settings
- `grid` - Grid strategy parameters (symbol, levels, price range)
- `risk_management` - Stop loss, take profit, drawdown limits, position sizing, and rebalancing thresholds
- `monitoring` - Logging and monitoring settings

**Configuration comments provide guidance:**
- Available options for each parameter
- Conservative vs aggressive recommendations
- Risk/reward trade-offs explained

## Running the System

**Simple Bot Execution:**
```bash
# Auto-discover and run first active config
uv run src/run_bot.py

# Run specific config
uv run src/run_bot.py bots/btc_conservative.yaml

# Validate configuration only
uv run src/run_bot.py --validate
```

**Configuration Management:**
```bash
# Check which config will be auto-discovered
ls bots/*.yaml

# Edit configuration
# Set active: true to enable, active: false to disable
```

## Development Patterns & Style

### Code Style
- **NO COMMENTS** in code unless explicitly requested
- Follow existing patterns in the codebase
- Use type hints consistently
- Keep functions focused and single-purpose

### Architecture Patterns
- **Interface-based design** - Clear separation between business logic and implementation
- **Dependency injection** - Adapters injected into strategies and engines
- **Event-driven** - WebSocket events trigger strategy decisions
- **Async/await** - Non-blocking I/O for real-time operations

### Error Handling
- Use custom exceptions from `utils/exceptions.py`
- Graceful degradation for network issues
- Comprehensive logging at appropriate levels
- Clean shutdown on signals (SIGINT, SIGTERM)

### Testing Requirements
- **Validate against Hyperliquid testnet** using provided test private key
- **Test all learning examples** ensure they work with real API responses
- **Configuration validation** verify all parameters actually work
- **Integration testing** test end-to-end trading workflows

### Debugging Guidelines

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kallie45s/hyperliquid-trading-bot](https://github.com/kallie45s/hyperliquid-trading-bot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
