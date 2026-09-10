---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

N42 is a **partial fork of [reth](https://github.com/paradigmxyz/reth)**. Most of reth is consumed as a
git dependency pinned to a tag (`reth = { git = "...", tag = "v1.11.0" }` — grep `Cargo.toml` for the tag
to confirm the current version); a subset of reth crates is vendored under `crates/` and substituted into
the whole dependency graph via `[patch.'https://github.com/paradigmxyz/reth.git']` in the root `Cargo.toml`.
N42-specific code lives under `crates/n42/` and `bin/n42/`.

Consensus is **APoS** (an extended Clique/PoA) plus a beacon/validator layer, not Ethereum's beacon chain.

## Build / test / lint

```bash
cargo build --release                  # builds default-members = bin/n42 only
cargo build --profile maxperf          # fat LTO, codegen-units=1
cargo check --workspace                # what CI gates on (.github/workflows/test.yml)
cargo test -p n42-testing              # the main test suite CI runs
cargo clippy --workspace --lib --bins --examples   # CI lint (.github/workflows/lint.yml)
```

Running one test:

```bash
cargo test -p n42-testing test_single_signer__no_votes_cast -- --nocapture
cargo test -p n42-clique --lib integration_tests::                # APoS unit/integration tests
```

Ethereum Foundation spec tests (`crates/n42/ef-tests`) are `#[ignore]`d by default and need fixtures:

```bash
EF_TESTS_PATH=/path/to/execution-spec-tests/fixtures \
  cargo test -p n42-ef-tests --test blockchain_tests -- --ignored
```

End-to-end (needs release binaries + node/npm + `jq`):

```bash
cargo build --release && cargo build --release -p mobile-sdk --example mobile-sdk-test
./tests/e2e.sh
```

Running a node:

```bash
cargo run --release --bin n42 -- node --chain crates/chainspec/res/genesis/n42_devnet.json \
  --dev.consensus-signer-private-key 0x... --dev.block-time 4s --http --ws
```

Caveat: `tests/e2e.sh` passes `--chain n42-devnet`, but the vendored
`crates/ethereum/cli/src/chainspec.rs` currently maps only `mainnet|sepolia|holesky|hoodi|dev` and falls
through to `parse_genesis()` for anything else — the `n42`/`n42-devnet` names were dropped during a reth
re-sync even though `N42` and `N42_DEVNET` still exist in `crates/chainspec`. Pass a genesis JSON path:
`--chain crates/chainspec/res/genesis/n42_devnet.json`.

`n42_devnet.json` is the native chain's devnet: `"stateScheme": "qmdb"`, `"consensus": "hotstuff"`
with four dev validators (secrets in `n42_devnet_validators.json`, derived from a public seed — dev
only), every Ethereum fork through Osaka active at genesis, the Prague system contracts in the
alloc, and gov5's production-shaped consensus settings: a 1 ETH `devBlockReward` with a faucet
(paid as withdrawals), a 4096-key `committeePool` with 64 signers (every header links to the
parent's committee evidence through `parentBeaconRoot`), and `epochLength` 20. Both `--chain <path>` and the `N42_DEVNET` constant build its genesis header the same way, with
the QMDB root of the alloc; a genesis file that declares its own fork schedule is trusted over the
legacy `N42_HARDFORKS` list. The same file is what a gov5 node is initialised from (`n42 init`).
On a chain whose genesis names a `hotstuff` validator set `bin/n42` runs `HotStuffConsensus`
(gov5's header profile and roots) instead of APoS and spawns no miner: the fleet
(`cargo run -p n42-h2-node --example h2_validator -- --chain <genesis> --propose …`) drives it over
the Engine API. `scripts/devnet-fleet.sh <tag> <secs> [--gov5]` runs the whole devnet — one QMDB
node, four Rust validators, or three plus a gov5 member from `../N42-gov5` (built from a commit that includes gov5 `95d47b46`,
the zero parent-beacon-root fix; `docs/gov5-cancun-parent-beacon-root.patch` is the same change for
an older branch). `GOV5_DELAY=<s>` starts the Go member late; `LATE_VALIDATOR=<i> LATE_DELAY=<s>`
starts one Rust member late with its own fresh execution layer, so it has to pull the chain by range;
`ABSENT_VALIDATOR=<i> ABSENT_AT=<s> ABSENT_FOR=<s>` runs one member with its own execution layer from
the start, kills both at `ABSENT_AT` keeping the datadirs, and restarts them `ABSENT_FOR` seconds later
(logs get a `-back` suffix) — the long-absence rejoin measurement;
`LATE_SNAPSHOT=<dir>` initialises that execution layer at a chain's head instead
(`n42-init-snapshot init`, from gov5's `n42-reth-state-dump` JSONL + header and a portable QMDB
snapshot — `n42-qmdb-export` or `n42-init-snapshot export`), so it pulls only what came after.
Every validator gets `--el-rpc <url>` (the execution layer's public RPC): it gossips its pool's
transactions on gov5's `transaction_v2` topic and hands gossiped ones to `eth_sendRawTransaction`
(`examples/send_tx` submits signed transfers to either client's RPC for this).
`n42-gov5-genesis` folds gov5's chainspec + alloc + genesis-block fields into a genesis this node
loads and checks the hash; `crates/chainspec/res/genesis/gov5/` holds chain 94 and 95 (see its
README for the `--fork-time` step before running on them). `docs/N42_26_PORT.md` "Joining a Go fleet"
lists every cross-client rule that had to be matched.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [n42blockchain/N42-rs](https://github.com/n42blockchain/N42-rs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
