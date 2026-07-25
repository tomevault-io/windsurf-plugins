---
trigger: always_on
description: <!-- Keep this file in sync with CLAUDE.md. Both describe the same codebase conventions; changes to one should be reflected in the other. -->
---

# GitHub Copilot Instructions — Helius Rust SDK

<!-- Keep this file in sync with CLAUDE.md. Both describe the same codebase conventions; changes to one should be reflected in the other. -->

Rust SDK for Helius APIs and Solana development. Built on `solana-client` and `tokio`, with async-first design and modular architecture. For full contributor details see [CLAUDE.md](../CLAUDE.md) and [CONTRIBUTIONS.md](../CONTRIBUTIONS.md).

## Critical: Use the Custom `Result<T>` Type

All fallible functions must use the SDK's `Result<T>` alias — never write `std::result::Result<T, HeliusError>` directly:

```rust
// In SDK source code (src/)
use crate::error::Result;

// In examples and tests
use helius::error::Result;

pub async fn my_function(&self) -> Result<Asset> { ... }
```

## Critical: Serde Conventions

JSON structs must use `camelCase` renaming to match the structure of Helius APIs. Always skip serializing `None` fields:

```rust
#[derive(Serialize, Deserialize, Debug, Clone)]
#[serde(rename_all = "camelCase")]
pub struct MyRequest {
    pub owner_address: String,
    #[serde(skip_serializing_if = "Option::is_none")]
    pub display_options: Option<DisplayOptions>,
}
```

If a struct only needs field-level renames (not `rename_all`), use `#[serde(rename = "fieldName")]` on individual fields.

## Critical: Error Handling with `HeliusError`

All errors flow through the `HeliusError` enum defined in `src/error.rs`. Use `thiserror` derives and `#[from]` for conversions. Never panic — always propagate errors with `?`:

```rust
// Good
let config = Config::new(api_key, cluster)?;

// Bad — never panic in library code
let config = Config::new(api_key, cluster).unwrap();
```

## Async Patterns

The SDK uses `tokio` as its async runtime. All async functions return `Result`:

```rust
pub async fn fetch_data(&self) -> Result<Data> {
    let response = self.client.get(url).send().await?;
    // ...
}
```

Use `Arc` for shared state across async boundaries (e.g., `Arc<Config>`, `Arc<RpcClient>`).

## Code Style

- **Naming:** `snake_case` functions/variables, `PascalCase` types/structs, `SCREAMING_SNAKE_CASE` constants
- **Line width:** 120 characters max (`rustfmt.toml` enforces this)
- **Imports:** Group related imports together, separated by blank lines. No strict ordering is enforced across the codebase
- **Cloning:** Avoid unnecessary `.clone()`. Use references where possible
- **Mutability:** Immutable by default. Only use `mut` when reassignment is required
- **Documentation:** `///` doc comments on all public items, with usage examples for complex functionality
- **Filenames:** Use underscores, not dashes. Do not name files `main.rs` — use descriptive names

## Project Structure

```
src/
  client.rs                         # Helius struct — main entry point, constructors, accessors
  builder.rs                        # HeliusBuilder for flexible client configuration
  config.rs                         # Config struct (api_key, cluster, endpoints, custom_url)
  rpc_client.rs                     # RpcClient with embedded Solana client, DAS API, RPC V2 methods
  request_handler.rs                # HTTP request handling, SDK user agent
  error.rs                          # HeliusError enum, Result<T> alias
  factory.rs                        # HeliusFactory for multi-cluster client creation
  lib.rs                            # Module declarations and public re-exports
  enhanced_transactions.rs          # parse_transactions, parsed_transaction_history
  optimized_transaction.rs          # Helius Sender, smart transactions, priority fees
  wallet.rs                         # Wallet API (identity, balances, transfers, history, funding info)
  webhook.rs                        # Webhook CRUD and address management
  staking.rs                        # Stake account creation, delegation, withdrawal
  websocket.rs                      # Enhanced WebSocket (Geyser) streaming
  types/
    inner.rs                        # Core types (Asset, Cluster, request/response structs)
    enums.rs                        # TokenType, Interface, OwnershipModel, etc.
    options.rs                      # Request option structs
    enhanced_websocket.rs           # WebSocket subscription types
    enhanced_transaction_types.rs   # Enhanced transaction types
  utils/
    is_valid_solana_address.rs      # Address validation
    make_keypairs.rs                # Keypair generation helpers
    deserialize_str_to_number.rs    # Custom serde deserializer
examples/
  das/                              # DAS API examples (get_asset_batch, get_asset_proof_batch, etc.)
  enhanced/                         # Enhanced Transactions API examples (parse, history, filters)
  helius/                           # Helius-specific examples (config, RPC V2 methods)
  solana/                           # Standard Solana RPC examples (get_latest_blockhash)
  transactions/                     # Helius Sender and smart transaction examples (send_smart_transaction)
  wallet/                           # Wallet API examples (balances, identity, transfers, history, funding info)
  webhooks/                         # Webhook CRUD examples (create, edit, delete, append/remove)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [helius-labs/helius-rust-sdk](https://github.com/helius-labs/helius-rust-sdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
