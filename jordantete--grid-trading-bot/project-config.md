---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A cryptocurrency grid trading bot supporting backtesting, paper trading, and live trading. Uses CCXT for exchange integration, with an event-driven architecture and strategy pattern for extensibility. Python 3.12+, managed with `uv`. Packaged as a src layout (`src/grid_trading_bot/`) and installable via `pip install grid_trading_bot`.

## Common Commands

```bash
# Install dependencies
uv sync --all-extras --dev

# Run the bot (backtest mode by default via config)
uv run grid_trading_bot run --config config/config.json

# Alternative: run via python -m
uv run python -m grid_trading_bot run --config config/config.json

# Show version
uv run grid_trading_bot --version

# Run all tests with coverage (unit + integration, excludes sandbox)
uv run python -m pytest --cov=grid_trading_bot --cov-report=term

# Run a single test file
uv run python -m pytest tests/order_handling/test_order_manager.py

# Run a single test
uv run python -m pytest tests/order_handling/test_order_manager.py::TestClassName::test_method_name

# Integration tests (backtest E2E, no network needed)
uv run python -m pytest tests/integration/test_backtest_e2e.py -v

# Sandbox smoke tests (requires network, validates ccxt/aiohttp compatibility)
uv run python -m pytest -m sandbox -v

# Update snapshots after intentional behavior changes
uv run python -m pytest tests/integration/test_backtest_e2e.py --update-snapshots

# Lint and format (via pre-commit or directly)
uv run ruff check --fix .
uv run ruff format .

# Run pre-commit hooks
uv run pre-commit run --all-files

# Start monitoring stack (Grafana/Loki/Alloy)
docker-compose up -d
```

## Architecture

### Package Layout

The project uses a **src layout**: all source code lives under `src/grid_trading_bot/`. Tests stay at the repo root under `tests/`. Runtime config files are in `config/`.

### Entry Point

`cli.py` is the Click-based CLI entry point (registered as `grid_trading_bot` console script). It parses CLI args via the `run` subcommand, creates a `GridTradingBot` per config file, and runs them concurrently via `asyncio.gather`. In live/paper mode, `BotController` (CLI commands) and `HealthCheck` (system monitoring) run as concurrent tasks. Supports `python -m grid_trading_bot` via `__main__.py`.

### Core Modules

- **`core/bot_management/`** — Bot lifecycle: `GridTradingBot` (orchestrator), `EventBus` (pub/sub), `BotController` (CLI), `HealthCheck`, `NotificationHandler` (Apprise alerts)
- **`core/grid_management/`** — `GridManager` computes grid levels (arithmetic/geometric spacing), `GridLevel` is a state machine tracking each level's cycle (`READY_TO_BUY` → `WAITING_FOR_BUY_FILL` → `READY_TO_SELL` → etc.)
- **`core/order_handling/`** — `OrderManager` orchestrates order placement and pairing (buy fill → place sell above, sell fill → place buy below). `BalanceTracker` tracks fiat/crypto with reserved amounts. `OrderBook` maps orders to grid levels
- **`core/persistence/`** — SQLite state persistence (live mode only): `StatePersistenceService` (event-driven checkpoints on every order fill/cancel), `StateRecoveryService` (merge recovery on restart — loads DB state, reconciles with exchange, restores balances), `SQLiteStateRepository` (WAL-mode SQLite backend), `Serializers` (Order/GridLevel/Balance ↔ dict). DB path: `data/{BASE}_{QUOTE}/state_{config_hash[:8]}.db`
- **`core/services/`** — Exchange abstraction via `ExchangeInterface` ABC. `BacktestExchangeService` reads CSV/CCXT OHLCV data. `LiveExchangeService` uses CCXT Pro WebSockets
- **`core/order_handling/execution_strategy/`** — `OrderExecutionStrategyInterface` ABC. Backtest variant is instant; live variant has retry logic with exponential backoff and slippage handling
- **`core/validation/`** — `OrderValidator` for order quantity/price validation

### Strategy Layer

- **`strategies/grid_trading_strategy.py`** — Implements `TradingStrategyInterface`. Handles both backtest (iterate OHLCV rows) and live (WebSocket price stream) execution loops. Manages initial purchase (50% allocation), grid order initialization, take-profit/stop-loss
- **`strategies/trading_performance_analyzer.py`** — Computes ROI, max drawdown, Sharpe ratio, etc.
- **`strategies/plotter.py`** — Plotly visualization for backtest results

### Config & Utils

- **`config/`** — `ConfigManager` (accessor methods for nested JSON), `ConfigValidator` (schema validation), `TradingMode` enum, custom `ConfigError` exceptions
- **`utils/`** — Logging setup, config name generation, performance results I/O

### Key Design Patterns

- **Factory Pattern**: `ExchangeServiceFactory` and `OrderExecutionStrategyFactory` select implementations based on `TradingMode`
- **Event Bus**: Decoupled communication via events (`ORDER_FILLED`, `ORDER_CANCELLED`, `START_BOT`, `STOP_BOT`, `INITIAL_PURCHASE_DONE`, `GRID_ORDERS_INITIALIZED`). Components subscribe to events rather than calling each other directly
- **State Machine**: `GridLevel` transitions through `GridCycleState` enum states as orders are placed and filled

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jordantete/grid_trading_bot](https://github.com/jordantete/grid_trading_bot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->
