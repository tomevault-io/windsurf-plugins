---
trigger: always_on
description: solroute is a Rust DEX aggregator for Solana. 6 pure DEX crates parse on-chain account data and compute swap outputs through a unified `Market` trait. An aggregator crate loads all pools from RPC, finds multi-hop routes, and provides pricing/caching. An engine crate runs a persistent HTTP API with live gRPC streaming. No external APIs -- all data is on-chain.
---

# Repository Guidelines

## Project Overview

solroute is a Rust DEX aggregator for Solana. 6 pure DEX crates parse on-chain account data and compute swap outputs through a unified `Market` trait. An aggregator crate loads all pools from RPC, finds multi-hop routes, and provides pricing/caching. An engine crate runs a persistent HTTP API with live gRPC streaming. No external APIs -- all data is on-chain.

## Architecture

```
solroute-core          Market trait, shared types, constants
    ^
    |
    +-- raydium-amm-v4    Constant product AMM
    +-- raydium-clmm      Concentrated liquidity (Q64.64 sqrt_price)
    +-- meteora-damm      Dynamic AMM V1 (constant product + stable) and V2 (sqrt_price)
    +-- meteora-dlmm      Dynamic liquidity bins
    +-- pumpfun-amm       Bonding curve (virtual reserves)

solroute-aggregator    Pool loading, routing, pricing, caching, CLI
solroute-engine        HTTP API, AccountStore, PoolRegistry, cold_start, streaming
solroute        Root crate: re-exports all DEX crates
```

No DEX crate imports another DEX crate.

### Data Flow (pure DEX crates)

```
Raw account bytes --BorshDeserialize--> Pool model struct
                                            |
                                            v
                                     DexMarket::new(pool, address, balances)
                                            |
                                            v
                                     impl Market { ... }
                                       +-- calculate_output()
                                       +-- calculate_price_impact()
                                       +-- current_price()
                                       +-- is_active()
```

## Key Directories

```
solroute/
+-- bin/
|   +-- engine.rs                       # Engine binary: immediate serve, background cold start
+-- Cargo.toml                          # Workspace root
+-- src/lib.rs                          # Root crate: re-exports all DEX crates
+-- crates/
|   +-- core/src/
|   |   +-- traits.rs                   # Market trait, AccountDataProvider, calculate_output_live
|   |   +-- constants.rs                # WSOL, USDC, USDT, quote_priority, infer_mint_decimals
|   +-- raydium-amm-v4/src/lib.rs       # RaydiumAMMV4 + RaydiumAmmV4Market
|   +-- raydium-clmm/src/
|   |   +-- lib.rs                      # RaydiumCLMMPool + RaydiumClmmMarket
|   |   +-- tick_arrays.rs              # Tick array bitmap + PDA derivation
|   +-- meteora-damm/src/
|   |   +-- lib.rs                      # V1 MeteoraDAMMMarket + V2 MeteoraDAMMV2Market
|   |   +-- models.rs                   # Pool models for V1, V2
|   |   +-- utils.rs                    # derive_token_vault_address
|   +-- meteora-dlmm/src/lib.rs         # MeteoraDLMMPool + MeteoraDlmmMarket
|   +-- pumpfun-amm/src/lib.rs          # PumpfunAmmPool + PumpfunAmmMarket
|   +-- aggregator/src/                 # Pool loading, routing, pricing, caching, CLI
|   |   +-- loader.rs                   # Async RPC pool loading (all DEXs)
|   |   +-- cache.rs                    # Disk cache + PDA extraction
|   |   +-- router.rs                   # Multi-hop routing (live data via AccountDataProvider)
|   |   +-- price.rs                    # SOL/USD on-chain via CLMM sqrt_price
|   |   +-- pool_index.rs              # In-memory token-pair graph
|   |   +-- cli.rs                      # Progress bars + interactive REPL
|   |   +-- main.rs                     # CLI binary (solroute-cli)
|   +-- engine/src/                     # Persistent service (library only, binary in bin/)
|   |   +-- account_store.rs            # DashMap store, implements AccountDataProvider
|   |   +-- pool_registry.rs            # Swappable validation, vault-to-pool reverse index
|   |   +-- cold_start.rs              # Background: vault fetch, tick arrays, bin arrays, bitmap exts
|   |   +-- streaming.rs                # Yellowstone gRPC: live updates + vault re-validation
|   |   +-- api.rs                      # Axum HTTP: /quote, /price, /health
+-- tests/
|   +-- helpers/mod.rs                  # Shared Geyser test helpers
|   +-- trade_stream.rs                 # Live DEX swap streaming via Yellowstone gRPC
|   +-- creation_stream.rs              # Live token + pool creation streaming
|   +-- pool_financials.rs              # Live pool update streaming
|   +-- validate_prices.rs              # Price validation across DEXs
```

## Development Commands

```bash
cargo check                        # Type-check all workspace crates
cargo build                        # Build all workspace crates
cargo test --workspace --lib       # Run unit tests
cargo build --release --bin solroute-engine  # Build engine binary
cargo build --release -p solroute-aggregator  # Build aggregator CLI

# Run engine (persistent service with HTTP API)
RPC_URL="https://..." cargo run --release --bin solroute-engine

# Run aggregator CLI (interactive REPL)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [anshuman008/solana-dex-aggregator](https://github.com/anshuman008/solana-dex-aggregator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-14 -->
