---
trigger: always_on
description: This guide helps Claude instances understand and work with the Kabu MEV bot codebase effectively.
---

# Kabu MEV Bot - Claude Development Guide

This guide helps Claude instances understand and work with the Kabu MEV bot codebase effectively.

## Quick Start Commands

```bash
# Run tests
make test

# Pre-release checks (includes fmt, clippy, taplo, and udeps)
make pre-release

# Clean unused dependencies
make udeps

# Type checking and linting
make lint
make check
make clippy  # Run clippy on all targets including tests and benches

# Build the project
cargo build --release

# Run the main binary
cargo run -p kabu

# Run specific swap tests
make swap-test FILE=./testing/backtest-runner/test_18567709.toml
make swap-test-all  # Run all swap tests
```

## Architecture Overview

Kabu is an MEV (Maximum Extractable Value) bot built with Rust, using a component-based architecture (evolved from actor-based) for concurrent processing of blockchain data and arbitrage opportunities.

### Core Design Principles

1. **Component-Based Concurrency**: Each component runs independently with message-passing communication
2. **Type Safety**: Extensive use of Rust's type system for correctness
3. **Modular Design**: Clear separation between data types, business logic, and infrastructure
4. **Performance First**: Optimized for low-latency arbitrage detection and execution

### Key Components

1. **Component System** (`crates/core/components/`)
   - Message-passing concurrency model using tokio
   - Components communicate via broadcast channels
   - Builder pattern for component initialization
   - Key components:
     - StateChangeArbSearcherComponent: Finds arbitrage in state changes
     - SwapRouterComponent: Routes and encodes swap transactions
     - TxSignersComponent: Signs transactions with managed keys
     - Broadcast components: Submit to network/flashbots

2. **Type System** (split into three crates for modularity)
   - `crates/types/entities/` - Core blockchain entities
     - Block, Transaction, Account structures
     - Strategy configurations
     - Pool configurations
   - `crates/types/market/` - Market-related types
     - Token: ERC20 token with price data
     - Pool trait and implementations
     - Market: Pool and token registry
     - SwapDirection, SwapPath, SwapPathBuilder
   - `crates/types/swap/` - Swap execution types
     - Swap: Executable arbitrage transaction
     - SwapLine: Path with calculated amounts
     - SwapStep: Individual pool interaction

3. **Strategy Layer** (`crates/strategy/`)
   - `backrun/`: State change arbitrage detection
     - BackrunConfig: Strategy configuration
     - StateChangeArbSearcher: Main arbitrage finder
     - SwapCalculator: Profit calculation logic
   - `merger/`: Transaction optimization
     - SamePathMerger: Combines similar paths
     - DiffPathMerger: Merges different opportunities
     - ArbSwapPathMerger: Multi-path arbitrage

4. **Execution Layer** (`crates/execution/`)
   - `estimator/`: EVM simulation for gas and success estimation
   - `multicaller/`: Custom contract for batch operations
     - Deploy logic for multicaller contract
     - Encoding for complex swap sequences

5. **DeFi Integrations** (`crates/defi/`)
   - `pools/`: Protocol implementations
     - UniswapV2Pool, UniswapV3Pool
     - CurvePool with multiple implementations
     - MaverickPool, PancakeV3Pool
   - `market/`: Market state management
   - `preloader/`: Initial state loading
   - `health_monitor/`: Pool reliability tracking

6. **Node Interaction** (`crates/node/`)
   - WebSocket subscription management
   - Block and transaction streaming
   - State diff calculation

7. **Database Layer** (`crates/evm/db/`)
   - In-memory state cache
   - Fork database for simulations
   - State diff application

## Component Communication Pattern

```rust
// Typical component setup using builder pattern
let component = StateChangeArbSearcherComponent::new(backrun_config)
    .on_bc(&blockchain, &strategy)  // Wire blockchain channels
    .with_market(market.clone());    // Set shared state

component.spawn(executor)?;          // Spawn with task executor

// Or using the centralized builder
let kabu_context = KabuBuildContextBuilder::new(
    provider.clone(),
    blockchain,
    blockchain_state.clone(),
    topology_config.clone(),
    backrun_config.clone(),
    multicaller_address,
    db_pool.clone(),
    is_exex,
)
.with_pools_config(pools_config.clone())
.with_swap_encoder(swap_encoder.clone())
.build();

let components = KabuComponentsBuilder::new(&kabu_context)
    .with_market_components()
    .with_network_components()
    .with_monitoring_components()
    .build()
    .await?;
```

## Type System Organization

### types/entities
Core blockchain and configuration types:
- `Block`, `Transaction`, `Account`
- `StrategyConfig` trait and implementations
- Pool loading configurations

### types/market  
Market structure and routing:
- `Token`: ERC20 with decimals, price, categories
- `Pool` trait: Unified pool interface
- `Market`: Registry of tokens and pools
- `SwapDirection`: Token pair direction in pool
- `SwapPath`: Route through multiple pools
- `PoolId`: Various pool identification methods

### types/swap
Execution and profit tracking:
- `Swap`: Final transaction ready for execution
- `SwapLine`: Path with amounts and gas costs

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cakevm/kabu](https://github.com/cakevm/kabu) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
