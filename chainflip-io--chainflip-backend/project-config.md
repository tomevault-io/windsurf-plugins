---
trigger: always_on
description: Chainflip is a decentralised cross-chain swap protocol. Users can swap native assets between different blockchains (e.g. BTC to ETH) without wrapping, bridging, or trusting a centralised intermediary. The protocol is operated by a permissionless set of **Validators** who collectively manage vault wallets on each supported chain via threshold signature schemes (TSS).
---

# Chainflip Development Guidelines

## What is Chainflip?

Chainflip is a decentralised cross-chain swap protocol. Users can swap native assets between different blockchains (e.g. BTC to ETH) without wrapping, bridging, or trusting a centralised intermediary. The protocol is operated by a permissionless set of **Validators** who collectively manage vault wallets on each supported chain via threshold signature schemes (TSS).

### Supported Chains and Assets

Ethereum (ETH, FLIP, USDC, USDT, WBTC), Bitcoin (BTC), Arbitrum (ETH, USDC, USDT), Solana (SOL, USDC, USDT).

### How a Swap Works (High-Level)

1. **Deposit**: A user sends funds to a Chainflip deposit address (or vault) on the source chain.
2. **Witnessing**: Validator engines observe the deposit on the source chain and report it to the State Chain.
3. **Execution**: The State Chain executes the swap through its internal AMM. USDC is used as the intermediate/hub asset for most pairs.
4. **Egress**: The State Chain schedules an output transaction on the destination chain. Validators collaboratively produce a threshold signature, and the transaction is broadcast.

### Core Components

- **State Chain** (`state-chain/`): A Substrate-based (Polkadot SDK) blockchain that is the coordination layer. All protocol logic lives here as FRAME pallets — swap execution, liquidity pools, vault rotation, ingress/egress scheduling, validator auctions, governance, and more. This is the core of the codebase.
- **Chainflip Engine** (`engine/`): An off-chain process run by every Validator alongside their State Chain node. It watches external blockchains (witnessing deposits, tracking gas prices), participates in threshold signing ceremonies (via the multisig sub-crate), and broadcasts signed transactions. Communicates with the State Chain via its RPC/extrinsic interface.
- **Smart Contracts** (external repos): Vault contracts deployed on Ethereum, Arbitrum, and Solana that custody user deposits and execute egress payouts under the authority of the Validator set's aggregate key.
- **FLIP Token**: The native staking and governance token (ERC-20 on Ethereum, also represented on the State Chain). Validators must bond FLIP to participate in the active set. The `cf-flip` pallet manages the on-chain representation; `cf-funding` handles staking.

### Key Concepts

- **Validator Set / Auctions**: Validators compete by staking FLIP. The top N stakers form the active **Authority Set** that manages vaults and processes swaps. Authority sets rotate on a regular cadence (`cf-validator` pallet). Delegators can contribute stake to groups of Validators managed by so-called Operators.
- **Vault Rotation**: When the authority set changes, the aggregate key for each chain's vault is rotated via a key generation ceremony, and the new key is activated and the old one deactivated. In some cases (BTC) this requires migrating fund to a new vault address (`cf-vaults` pallet).
- **Threshold Signing (TSS)**: Validators use multi-party computation to jointly sign transactions without any single party holding the full private key (`cf-threshold-signature` pallet + `engine/multisig/`).
- **Witnessing / Elections**: Validators observe external chain events and reach consensus on what happened via the elections framework (`cf-elections` pallet). This covers deposits, gas price updates, and other chain-tracking data.
- **Liquidity Pools & AMM**: The `cf-pools` pallet implements a novel concentrated-liquidity AMM with limit orders layered on top. Liquidity providers (LPs) deposit assets and set price ranges. The `cf-swapping` pallet routes swaps through the pool(s) and handles features like DCA (chunked execution) and cross-chain messaging (CCM).
- **Ingress / Egress**: `cf-ingress-egress` manages deposit channels (addresses where users send funds) and egress scheduling (batching and sending output transactions).
- **Broadcast**: `cf-broadcast` manages the lifecycle of outgoing transactions on external chains — from threshold signing through to confirmation.

## Build/Test/Lint Commands

- Build: `cargo build --release` or `cargo build -p <package>`
- Lint: `cargo check` or `cargo cf-clippy`
- Lint package: `cargo check -p <package>`
- Format: `cargo fmt --all` (never `cargo fmt -- <filename>`: the per-file form ignores the crate's edition and applies the wrong import-ordering style)
- Run all tests: `cargo nextest run`
- Run package tests: `cargo nextest run -p <package>`
- Run single test: `cargo nextest run <test_name>` or `cargo nextest run <module>::<test_name>`
- Show test output: Add `-- --nocapture` to test commands
- Clean build: `cargo clean` or `cargo clean -p <package>`

## Code Style Guidelines

- Formatting: All formatting rules are imposed by `cargo fmt --all`, run this before every commit.
- Errors: Use `Err(anyhow!("message"))` at end of functions, `bail!()` for early returns
- PRs: Keep small (<400 lines), organize meaningful commits

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chainflip-io/chainflip-backend](https://github.com/chainflip-io/chainflip-backend) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
