---
trigger: always_on
description: This file provides guidance to AI coding agents when working with code in this repository.
---

# AGENTS.md

This file provides guidance to AI coding agents when working with code in this repository.

## Project Overview

Midnight Node is a Substrate-based blockchain implementation for the Midnight network - a privacy-preserving blockchain with zero-knowledge proof capabilities. It operates as a Cardano Partner Chain with integration to the Cardano mainchain.

## Build Commands

**Daily development:** `cargo check`, `cargo test`, `cargo clippy`, `cargo fmt`, `cargo build --release`

**Run specific test:** `cargo test test_name` or `cargo test -- --nocapture` for output

**Earthly commands:**
```bash
earthly -P +rebuild-metadata              # Update runtime metadata
earthly -P +rebuild-chainspec --NETWORK=<network>  # Rebuild chainspec for network
earthly -P +rebuild-all-chainspecs        # Rebuild all chainspecs
earthly -P +rebuild-genesis-state-<NETWORK>  # Rebuild genesis for specific network
earthly -P +rebuild-all-genesis-states    # Rebuild all network genesis states
earthly +node-image                       # Build node Docker image
earthly +toolkit-image                    # Build toolkit image
earthly doc                               # List all available targets
```

**GitHub PR bots:** Comment on a PR to trigger rebuilds:
- `/bot rebuild-metadata` - Rebuild runtime metadata
- `/bot rebuild-chainspec <network1> <network2>` - Rebuild chainspecs for specified networks
- `/bot cargo-fmt` - Run cargo fmt

**E2E tests (just):**
```bash
just toolkit-e2e <NODE_IMAGE> <TOOLKIT_IMAGE>
```

**Genesis generation:**
```bash
./scripts/genesis/genesis-construction.sh  # Interactive genesis construction wizard
```
See [Genesis Construction Guide](docs/genesis/construction.md) for complete documentation.

**Genesis verification:**
```bash
./scripts/genesis/genesis-verification.sh  # Interactive genesis verification wizard
```
See [Genesis Verification Guide](docs/genesis/verification.md) for complete documentation.

## Architecture

```
/node/         - Main node binary, CLI, RPC server, service initialization
/runtime/      - Substrate runtime assembly, pallet configuration
/pallets/      - Custom runtime pallets:
  ├── midnight             - Core ledger state and transaction execution
  ├── midnight-system      - System transaction management (root privileges)
  ├── cnight-observation   - Cardano bridge (cNIGHT to DUST token bridging)
  ├── federated-authority  - Multi-collective governance system
  ├── federated-authority-observation - Governance sync from mainchain
  └── version              - Runtime version tracking in block digests
/primitives/   - Shared types and runtime interfaces (7 crates)
/ledger/       - Midnight ledger types and state management
/res/          - Chain specifications and network configuration
/res/cfg/      - Config presets per network
/util/toolkit/ - Transaction generator for testing
/tests/e2e/    - End-to-end test suite
```

**Consensus:** AURA (6-second blocks) + GRANDPA (finality) + BEEFY (bridge security)

**Key dependencies:**
- `midnight-ledger` - Privacy ledger with zero-knowledge proofs
- `polkadot-sdk` - Substrate framework
- `partner-chains` - Cardano sidechain framework

## Development Setup

```bash
source .envrc  # Load environment with direnv
cargo check
```

See `docs/rust-setup.md` for Rust toolchain installation.

**Running a local node** (always use release mode):
```bash
cargo build --release
CFG_PRESET=dev ./target/release/midnight-node
```
Ports: P2P 30333, RPC 9944

**Debugging ledger issues:** Keep a local checkout of `midnight-ledger` for searching error messages and understanding `LedgerState` implementation.

**Recommended tools:**
- [gh CLI](https://cli.github.com/) - GitHub CLI for creating PRs, viewing issues, etc.

## When to Rebuild

**Metadata** (use `/bot rebuild-metadata` on PR, or `earthly -P +rebuild-metadata`):
- Pallet storage items change
- Extrinsic signatures change
- Runtime APIs are added/modified

**Genesis** (`earthly -P +rebuild-genesis-state-<NETWORK>`):
- Genesis code changes in toolkit
- Genesis seeds change
- New ledger version

## Network Configurations

Config presets are in `res/cfg/`:
- `dev` - Local development (no AWS secrets required)
- `qanet` - QA testing network
- `preview` - Preview/staging network
- `preprod` - Pre-production network

Networks other than `dev` require AWS access for genesis rebuilds. Contact the node team if you need help.

## Git Workflow

**Branching:** Always create a new branch for changes - never push directly to main. Branch names should be prefixed with a short name moniker (e.g., `jill-my-feature`).

**Commit messages:** Must follow [Conventional Commits](https://www.conventionalcommits.org/) format:
- `feat:` new features
- `fix:` bug fixes
- `chore:` maintenance tasks
- `docs:` documentation changes
- `refactor:` code refactoring
- `test:` adding/updating tests

**Do not include:**
- LLM watermarks (e.g., "Generated by Claude", "Written by AI", etc.)
- Co-authored-by lines for LLM tools (e.g., `Co-Authored-By: Claude <noreply@anthropic.com>`)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [midnightntwrk/midnight-node](https://github.com/midnightntwrk/midnight-node) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
