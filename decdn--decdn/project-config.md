---
trigger: always_on
description: Decentralized CDN (deCDN) — nodes cache and serve content-addressed blobs over iroh QUIC, clients pay per-MB via off-chain USDC shared payment pools. Rust implementation; the initial network deployment targets tens of nodes on an Arbitrum Sepolia testnet. "PoC" in code and ADR comments refers to that network-scale milestone, not contract-surface scope — the on-chain surface ships at full production shape with governance-tunable economics from day one (see [ADR 016 § Contract Inventory](adr/016-c
---

# CLAUDE.md

## Project Overview

Decentralized CDN (deCDN) — nodes cache and serve content-addressed blobs over iroh QUIC, clients pay per-MB via off-chain USDC shared payment pools. Rust implementation; the initial network deployment targets tens of nodes on an Arbitrum Sepolia testnet. "PoC" in code and ADR comments refers to that network-scale milestone, not contract-surface scope — the on-chain surface ships at full production shape with governance-tunable economics from day one (see [ADR 016 § Contract Inventory](adr/016-contract-interactions.md) and [§ Tunable Economics](adr/016-contract-interactions.md#tunable-economics)).

**Status: Early implementation.** Cargo workspace with 11 crates and two binaries (#421): the `node` crate builds the `decdn-node` daemon (runtime bring-up, admin RPC server, dispatch limiter, probe handler); the `cli` crate builds the user-facing `decdn` binary (`key-gen`, `whoami`, `config {…}`, `probe`, `fetch`, `setup`, `node {…}`, `bundle {pull}`, `origin {import}`, `pool {…}`, `publish {…}`, `appeal {slash}`). See the [Crate Structure](#crate-structure) section for what each crate owns. No crate is a stub.

**Pre-launch: wire-breaking changes are fine.** Nothing is deployed and there are no live peers. Do not add backward-compatibility shims, version negotiation, dual-format readers, or migration paths for wire, postcard, ABI, config, or storage changes. Change the format, update every side in the same PR, and delete the old shape. Compatibility work only becomes real after the first public deployment.

See [CONTRIBUTING.md](CONTRIBUTING.md) for build commands, ADR conventions, pre-commit hooks, and development environment setup.

**Present-tense canon, never changelog voice.** Docstrings, comments, and ADRs describe current behavior as it is. No "replaced the old X", "used to…", "re-homed from", dates, or "supersedes". Every line must stand alone read cold; history belongs in git, not the source.

**Working specs stay out of the repo.** Write specs to a scratch location while you work; never commit them. Repos store no historical specs.

**ADRs.** ADRs live in `adr/`. They are the primary design artifacts. `adr/architecture.md` is the living overview. See [CONTRIBUTING.md](CONTRIBUTING.md) for ADR conventions.

- The next ADR number is 042. Name each file `NNN-topic.md`. Use a 3-digit prefix.
- List `adr/` and find the highest number before you make a new ADR. Do not trust this note for the current number.
- Do not use these numbers again: 004, 006, 007, 010, 015, 020, 021, 023, 025, 027, 029, 031, 032, 033, 034, 035. Each one is retired or reclassified.
- Retired ADRs move to `adr/_history/`. Read the file there if you need the history. Do not add the history to this file.
- Write ADRs in ASD-STE100 Simplified Technical English: short sentences, active voice, present tense, one idea per sentence. Every ADR follows this. Keep new ADRs and edits the same.

## Common Commands

```bash
cargo build && cargo clippy          # build + lint (clippy is the usual CI failure)
cargo nextest run                    # test (preferred over cargo test)
cargo nextest run -p decdn-protocol  # single crate
cargo fmt -- --check                 # check formatting
cargo deny check                     # license + advisory audit
pre-commit run --all-files           # run all hooks
# Contracts — mirror what CI runs.
(cd contracts && forge fmt --check && FOUNDRY_PROFILE=ci forge build --sizes --deny warnings && forge test)
(cd contracts && aderyn -o /tmp/aderyn.md --no-snippets --skip-update-check)  # fail-on: high
(cd contracts && slither . --config-file slither.config.json)                 # fail-on: medium
```

Full Solidity workflow, CI gotchas, static analysis, coverage, and gas snapshots live in [CONTRIBUTING.md § Solidity development](CONTRIBUTING.md#solidity-development). CI fails on warnings that local `forge test` ignores, so reproduce with the `FOUNDRY_PROFILE=ci` commands before pushing.

## Architecture

**Language:** Rust (edition 2024, MSRV 1.95). **Networking:** iroh (QUIC transport, NAT traversal, content-addressed blobs).

**Code style:** `rustfmt.toml` sets `max_width = 100`.

**Anti-panic policy:** Clippy denies `unwrap_used`, `expect_used`, `panic`, `indexing_slicing`, `todo`, and `unimplemented` workspace-wide. Use `Result`/`Option` combinators or `.get()` for indexing. This is the most common CI failure for new code.

**Numeric safety:** `cast_possible_truncation`, `cast_sign_loss`, and `cast_precision_loss` are `deny`, not `warn`. A narrowing or sign-changing cast needs a per-site `#[allow]`/`#[expect]` with a comment saying why it cannot lose data.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [decdn/decdn](https://github.com/decdn/decdn) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
