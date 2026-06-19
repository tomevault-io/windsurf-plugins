---
trigger: always_on
description: **tronz** is an async-first Rust SDK for the TRON blockchain, inspired by [alloy](https://github.com/alloy-rs/alloy). It provides ergonomic APIs for TRON native operations (TRX transfer, Stake 2.0, delegation) and EVM-compatible smart contracts (TRC20/TRC721).
---

# tronz — Claude Code Context

## Project Overview

**tronz** is an async-first Rust SDK for the TRON blockchain, inspired by [alloy](https://github.com/alloy-rs/alloy). It provides ergonomic APIs for TRON native operations (TRX transfer, Stake 2.0, delegation) and EVM-compatible smart contracts (TRC20/TRC721).

- **Status:** v0.1.1 — active development, not yet production-ready
- **Rust edition:** 2024, MSRV 1.85 (required for stable RPITIT)
- **License:** MIT OR Apache-2.0
- **Repo:** https://github.com/throgxyz/tronz

---

## Workspace Layout

```
tronz/
├── Cargo.toml                  # Workspace root (members: crates/*)
├── Makefile                    # Local CI targets (make ci, make clippy, etc.)
├── deny.toml                   # cargo-deny config (licenses, advisories, bans)
├── typos.toml                  # typos config (excludes proto/, custom word list)
├── DESIGN.md                   # Architecture design doc (1600+ lines)
└── crates/
    ├── primitives/             # tronz-primitives: leaf types, no I/O
    ├── signer/                 # tronz-signer: TronSigner trait + LocalSigner
    ├── provider/               # tronz-provider: transport + domain model + provider
    ├── contract/               # tronz-contract: TRC20/TRC721 ABI bindings
    └── tronz/                  # umbrella crate (re-exports everything)
```

Examples live in a separate repo: https://github.com/throgxyz/examples

### Crate dependency graph

```
tronz-primitives  (leaf: Address, Trx, ResourceCode, RecoverableSignature)
      ↑        ↑
tronz-signer   tronz-provider  (domain types + proto codec + gRPC transport + provider)
                    ↑
              tronz-contract    (sol! bindings, TRC20/TRC721, call/deploy builders)
                    ↑
                  tronz         (umbrella re-export)
```

---

## Key Design Decisions

### 1. RPITIT (Return-Position Impl Trait in Traits)
All async traits (`TronSigner`, `TronTransport`, `TronProvider`) use stable RPITIT instead of `async_trait`. This gives zero-cost futures (no `Box<dyn Future>` heap allocation). Requires Rust 1.85+.

### 2. gRPC Transport (not JSON-RPC)
TRON nodes expose a gRPC `WalletClient` service. tronz uses `tonic` + `prost` directly. There is no JSON-RPC equivalent for TRON (unlike Ethereum).

**Default endpoints:**
- Mainnet: `https://grpc.trongrid.io:443` (`TRONGRID_MAINNET`)
- Nile testnet: `http://grpc.nile.trongrid.io:50051` (`TRONGRID_NILE`)

### 3. Proto Types Are Private
Prost-generated code lives in `provider/src/proto/` (private module). All public APIs use domain types (`RawTransaction`, `AccountInfo`, `BlockInfo`). Codec conversions are hidden inside `transport/grpc/codec.rs`.

### 4. Filler Chain (Adapted from Alloy)
```
provider.send_trx().to(addr).amount(trx).send()
  → TaposFiller: fetch latest block, fill ref_block_*, expiration, timestamp
  → FeeLimitFiller: set fee_limit for smart contract calls
  → SignerFiller: sign tx_id with LocalSigner
  → broadcast_transaction()
  → PendingTransaction { tx_id }
```
Fillers are composed via `JoinFill<L, R>`. `ProviderBuilder::with_recommended_fillers()` chains `TaposFiller + FeeLimitFiller`.

### 5. Two Transaction Build Paths
- **Native contracts** (transfer, freeze, delegate…): Client builds `RawTransaction` locally, TaposFiller fills TAPOS fields, client signs and broadcasts.
- **Smart contracts** (`TriggerSmartContract`, `CreateSmartContract`): Client sends params to node → node builds `RawTransaction` → client signs locally → broadcasts. TaposFiller skips if `ref_block_bytes` is already set.

### 6. Extension Trait Pattern (Trc10Api, etc.)
Extra functionality that doesn't belong on `TronProvider` directly lives in extension traits under `provider/src/ext/`. Import the trait to unlock the methods:
```rust
use tronz::providers::ext::Trc10Api as _;
provider.issue_trc10().name("MTK").send().await?;
```

### 7. Alloy Integration Strategy
| What | How |
|------|-----|
| Primitive types (B256, U256, Bytes) | Direct dep on `alloy-primitives` |
| TRC20/TRC721 ABI codec | `alloy-sol-macro` (`sol!`) + `alloy-sol-types` |
| Dynamic ABI | `alloy-dyn-abi` + `alloy-json-abi` |
| Provider/Transport/Network traits | NOT reused (TRON-specific) |
| ProviderBuilder/TxFiller/JoinFill | Adapted pattern, not the same code |
| Module visibility | Private `mod` + `pub use` re-exports (same as alloy-contract) |

### 8. TRON Address Format
```
TRON address = 0x41 || EVM-body (20 bytes) = 21 bytes total
Base58check:  "T..." (user-facing)
Hex:          "41..." or "0x41..."

FromStr auto-detects: starts with 'T' → base58, else → hex
```

---

## Module Map (tronz-provider internals)

```
provider/src/
├── types/          # Public domain model (no proto leakage)
│   ├── transaction.rs    # RawTransaction, SignedTransaction, TransactionRequest
│   ├── block.rs          # BlockInfo + TAPOS extraction helpers
│   ├── account.rs        # AccountInfo, AccountResource, DelegatedResource
│   ├── contract.rs       # ContractType enum + param structs (incl. AssetIssueContract)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [throgxyz/tronz](https://github.com/throgxyz/tronz) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-18 -->
