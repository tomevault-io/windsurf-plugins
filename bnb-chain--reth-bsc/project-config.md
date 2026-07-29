---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project shape

`reth-bsc` is **not** a fork of Reth. It is a downstream crate that re-uses Reth's `NodeBuilder` API to assemble a BSC-compatible client. Everything BSC-specific (Parlia consensus, BSC hardforks, system contracts, PoSA mining, MEV/Parlia/Miner RPCs, EVN peer features, BSC precompiles) lives here; generic EL behavior comes from upstream Reth pinned by git `rev` in `Cargo.toml`.

- Workspace members: the root crate (binary `reth-bsc`, library `reth_bsc`) and `testing/bsc-ef-tests` (execution-spec tests harness).
- All `reth-*` deps are pinned to one commit in `Cargo.toml` (currently `bnb-chain/reth` rev `ef46a48…`). If you change the Reth rev, update **every** `reth-*` line — a mismatched rev produces duplicate-crate build failures. The `testing/bsc-ef-tests/Cargo.toml` uses `branch = "develop"` of the same fork; that's intentional but keep it aligned when bumping.
- `build.rs` scans `src/system_contracts/<hardfork>/{mainnet,chapel,rialto}/*` at build time and emits `src/system_contracts/embedded_contracts.rs` (a `phf_map` keyed as `"<hardfork>_<network>_<contract>"`). It also records the git SHA into `RETH_BSC_GIT_SHA` / `RETH_BSC_GIT_SHA_LONG` used at startup and in the P2P client string. If you add a new hardfork directory with system contracts, add it to the `hardforks` list in `build.rs` so cargo rebuilds when those files change.

## Common commands

Build:
```bash
cargo build                               # debug
cargo build --release                     # plain release
make build                                # release + jemalloc,asm-keccak + target-cpu=native
make maxperf                              # LTO=fat, single codegen-unit release
make bench-test                           # maxperf + `bench-test` feature (exposes engine_forkchoiceUpdatedV1)
```
The `jemalloc` feature is enabled by default and requires `tikv-jemallocator` on Unix; do not build with `--no-default-features` casually.

Lint / hygiene (match CI in `.github/workflows/ci.yml`):
```bash
cargo check
cargo clippy --workspace --tests --all-features     # CI runs with RUSTFLAGS="-D warnings"
cargo +nightly udeps --workspace --lib --examples --tests --benches --all-features --locked
cargo fmt                                           # nightly rustfmt settings in rustfmt.toml
```

Tests:
```bash
cargo test --all -- --test-threads=1                 # CI setting; many tests touch global OnceLocks / env
cargo test -p reth_bsc <test_name>                   # run a single test (substring match)
cargo test -p reth_bsc module::path::test_fn -- --exact --nocapture
```

Execution-spec tests (network-download fixtures):
```bash
make download-eest      # pulls EEST v5.4.0 fixtures into testing/bsc-ef-tests/execution-spec-tests
make ef-tests           # cargo test -p bsc-ef-tests --release --features ef-tests,jemalloc,asm-keccak
make ef-tests-nextest   # same, via cargo-nextest
make clean-eest
```

Run (binary is `reth-bsc`, chain ids `bsc`, `bsc-testnet`):
```bash
./target/release/reth-bsc node --chain bsc --datadir ./data_dir
./target/release/reth-bsc node --full --chain bsc --datadir ./data_dir            # full node
```

## Where the wiring lives

The non-obvious cross-cutting pieces — read these together when anything spans components:

- **`src/main.rs`** is the integration point. It parses `BscCliArgs`, then uses `Cli::<BscChainSpecParser, BscCliArgs>::parse().run_with_components::<BscNode>(...)`. The async closure:
  1. applies the genesis-hash override,
  2. hydrates the global `MiningConfig` (CLI > env > defaults) and loads the signing key from keystore / hex,
  3. initializes the global BLS signer (CLI > env),
  4. builds and stores `EvnConfig` (EVN peer-tx-broadcast policy),
  5. parses proxyed peer IDs,
  6. calls `.extend_rpc_modules(...)` to register BSC-only RPC namespaces (`parlia`, `mev`, `miner`, BSC eth-extensions, blob). It **removes** reth's built-ins `miner_setExtra`/`setGasPrice`/`setGasLimit` and `eth_coinbase` before registering our versions — don't re-add them upstream-style,
  7. sends the beacon engine handle back to the network via a oneshot and stores the engine-API mpsc sender globally.

- **`src/shared.rs`** holds nearly all cross-component globals as `OnceLock`s + a few `RwLock`s: snapshot provider, header/block-number accessors, engine-API sender, network handle, payload-events broadcast, bid-package queue, proxyed peer IDs, IPC client, miner runtime knobs, etc. Components register into these from different phases of startup (consensus module publishes the snapshot provider, then the miner waits up to ~10s for it — see `node::engine::BscPayloadServiceBuilder`). When adding a cross-phase dependency, extend `shared.rs` rather than threading handles through builders.

- **`src/node/mod.rs` → `BscNode`** composes: `BscPoolBuilder` (pool), `BscExecutorBuilder` (EVM), `BscPayloadServiceBuilder` (payload + miner bootstrap), `BscNetworkBuilder`, `BscConsensusBuilder`. `BscNodeAddOns` wires `BscEthApiBuilder` plus engine/payload validators (`src/node/engine_api/`).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bnb-chain/reth-bsc](https://github.com/bnb-chain/reth-bsc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
