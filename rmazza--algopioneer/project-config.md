---
trigger: always_on
description: This is a production-ready, enterprise-grade Rust-based algorithmic trading system named `algopioneer`. It is designed to interact with multiple cryptocurrency exchanges (Coinbase, Kraken) and equity brokers (Alpaca) for trading research and execution, supporting both live and paper trading modes with comprehensive risk management and observability.
---

# Project Overview

This is a production-ready, enterprise-grade Rust-based algorithmic trading system named `algopioneer`. It is designed to interact with multiple cryptocurrency exchanges (Coinbase, Kraken) and equity brokers (Alpaca) for trading research and execution, supporting both live and paper trading modes with comprehensive risk management and observability.

**Quality Score:** 9.9+/10 (Production-Ready)

**Key Technologies:**

*   **Language:** Rust (Edition 2021)
*   **Core Libraries:**
    *   `cbadv`: Coinbase Advanced Trade API client
    *   `apca`: Alpaca Markets API client for equities
    *   `polars`: High-performance data manipulation and analysis
    *   `ta`: Technical analysis indicators
    *   `tokio`: Asynchronous runtime for concurrent operations
    *   `axum`: HTTP server for health checks and metrics
    *   `opentelemetry`: Distributed tracing and observability
    *   `prometheus`: Metrics collection and exposure
    *   `dashmap`: Lock-free concurrent hash maps
    *   `rust_decimal`: High-precision financial calculations
    *   `num-decimal`: Additional decimal number handling for Alpaca integration
    *   `tokio-tungstenite`: WebSocket connections
    *   `tikv-jemallocator`: Optimized memory allocation for long-running processes
    *   `governor`: Rate limiting for API calls
    *   `thiserror`: Typed error handling
    *   `bytes`: Zero-copy string handling
    *   `uuid`: Trade ID generation for logging/recording
    *   `proptest`: Property-based testing (dev dependency)
    *   `aws-sdk-dynamodb` / `aws-config`: Optional AWS DynamoDB trade recording

**Architecture:**

Production-grade modular application with dependency injection, trait-based abstractions, and comprehensive error handling.

*   **Entry Point (`src/main.rs`):** CLI with subcommands for trading, backtesting, portfolio management, and pair discovery
*   **CLI (`src/cli/`):**
    *   `mod.rs`: CLI struct and command definitions (Clap-based)
    *   `config.rs`: CLI configuration structs (`SimpleTradingConfig`, `DualLegCliConfig`)
*   **Commands (`src/commands/`):**
    *   `mod.rs`: Command re-exports
    *   `trade.rs`: Moving Average strategy trade handler
    *   `backtest.rs`: Backtesting evaluation handler
    *   `dual_leg.rs`: Dual-leg (basis/pairs) trading handler
    *   `portfolio.rs`: Multi-pair portfolio trading handler
    *   `discover.rs`: Pair discovery command handler
*   **Trading (`src/trading/`):**
    *   `mod.rs`: Trading module re-exports
    *   `simple_engine.rs`: Simple trading engine for Moving Average strategy
*   **Strategies (`src/strategy/`):**
    *   `mod.rs`: Strategy trait definitions and re-exports
    *   `dual_leg_trading.rs`: Main dual-leg arbitrage strategy with state machine, recovery system, and circuit breaker
    *   `moving_average.rs`: Moving Average Crossover strategy
    *   `supervisor.rs`: Generic strategy supervisor with panic recovery, PnL aggregation, and health monitoring
    *   `tick_router.rs`: Lock-free market data distribution with backpressure handling
*   **Exchange Abstraction (`src/exchange/`):**
    *   `mod.rs`: Exchange-agnostic traits (`Executor`, `ExchangeClient`, `WebSocketProvider`) and factory functions
    *   `coinbase/`: Coinbase-specific implementation (client, websocket, market data provider)
    *   `kraken/`: Kraken-specific implementation (client, websocket, market data provider)
    *   `alpaca/`: Alpaca-specific implementation
        *   `alpaca_client.rs`: Alpaca API client with order execution and position tracking
        *   `websocket.rs`: Real-time equity market data streaming via Alpaca WebSocket
        *   `utils.rs`: Conversion utilities for Alpaca types
*   **Coinbase Integration (`src/coinbase/`):**
    *   `mod.rs`: API client with position querying
    *   `websocket.rs`: Real-time market data streaming
    *   `market_data_provider.rs`: Abstracted data sources (live + synthetic)
*   **Risk Management (`src/risk/`):**
    *   `mod.rs`: Risk module re-exports
    *   `daily_limit.rs`: Daily loss limit enforcement (`DailyRiskEngine`)
    *   `executor.rs`: Risk-aware trade executor wrapper
*   **Orders (`src/orders/`):**
    *   `mod.rs`: Order module re-exports
    *   `tracker.rs`: Order state tracking with TOCTOU-safe operations
    *   `reconciler.rs`: Position reconciliation after network failures
    *   `types.rs`: Order-related types (OrderRecord, ExecutionReport, etc.)
*   **Discovery (`src/discovery/`):**
    *   `mod.rs`: Pair discovery and optimization module
    *   `config.rs`: Discovery configuration
    *   `filter.rs`: Candidate pair filtering with cointegration tests
    *   `optimizer.rs`: Grid search parameter optimization
    *   `sector.rs`: Sector classification for pair grouping
    *   `error.rs`: Typed discovery errors
*   **Logging (`src/logging/`):**
    *   `mod.rs`: Trade recording module re-exports
    *   `recorder.rs`: Base trade recording logic

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rmazza) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
