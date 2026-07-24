---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Origin DeFi's OTokens monorepo containing smart contracts for:
- **OUSD** (Origin Dollar) - a yield-bearing stablecoin
- **OETH** (Origin Ether) - an Ethereum liquid staking token
- **OS** (Origin Sonic) - Sonic chain native token

Deployed on Ethereum Mainnet, Base, Arbitrum, Sonic, Plume, Hoodi, and HyperEVM. All smart contract work happens in the `contracts/` directory.

## Setup

```bash
cd contracts
cp dev.env .env          # Set PROVIDER_URL to Alchemy/Infura endpoint
pnpm i
```

Key `.env` variables: `PROVIDER_URL`, `SONIC_PROVIDER_URL`, `BASE_PROVIDER_URL`, `BLOCK_NUMBER`, `ACCOUNTS_TO_FUND`.

## Commands (run from `contracts/`)

### Build
```bash
pnpm hardhat compile           # Compile changed contracts
pnpm clean && pnpm hardhat compile  # Full recompile
```

### Lint & Format
```bash
pnpm lint                      # Run all linters (Solidity + JS)
pnpm lint:sol                  # solhint for Solidity
pnpm lint:js                   # eslint for JavaScript
pnpm prettier:check            # Check formatting
pnpm prettier                  # Format all files
```

### Unit Tests
```bash
pnpm test                      # Mainnet unit tests
pnpm test:base                 # Base network unit tests
pnpm test:sonic                # Sonic network unit tests
pnpm test:coverage             # Mainnet unit tests with coverage
pnpm test test/**/FILE_NAME.js # Running a specific test file
```

### Fork Tests (require `PROVIDER_URL` in `.env`)
```bash
# Option 1: Fork each time (like CI)
pnpm test:fork                                    # All mainnet fork tests
pnpm test:fork -- test/strategies/foo.fork-test.js  # Single fork test file

# Option 2: Nested forking (faster for dev iteration)
FORK=true pnpm run node        # Terminal 1: start forked node with deployments
pnpm test:fork                 # Terminal 2: tests reuse running node

# Other networks
pnpm test:arb-fork
pnpm test:base-fork
pnpm test:sonic-fork
pnpm test:hol-fork
```

### Useful Options
```bash
export DEBUG=origin:*          # Enable all debug logging
export REPORT_GAS=true         # Show gas usage in test output
export CONTRACT_SIZE=true      # Show contract sizes after compile
```

## Workflow Guidance

- Run repo commands from `contracts/` for smart contract work.
- After making code changes, run Prettier before finishing.
- For JS edits under `contracts/`, run `pnpm prettier:js`.
- For Solidity edits under `contracts/`, run `pnpm prettier:sol`.
- If both JS and Solidity files changed, run both commands.
- Prefer the smallest relevant verification after edits.
- Do not reformat or modify unrelated files just to satisfy style.
- Do not fix unrelated failing tests or lint issues unless explicitly asked.

## Architecture

### Core Pattern: Upgradeable Proxy Contracts
All major contracts use the OpenZeppelin upgradeable proxy pattern. Each has a `*Proxy` contract (minimal proxy) pointing to an implementation. Proxies are deployed via `hardhat-deploy` scripts in `deploy/`.

### Vaults (Central Component)
Vaults (`contracts/vault/`) are the core of each OToken. They handle:
- Minting/burning OTokens
- Managing collateral allocation across strategies
- Rebalancing via `allocate()`
- Yield accounting via `rebase()`

Each chain/token has its own vault: `OUSDVault`, `OETHVault`, `OETHBVault` (Base), `OETHSVault` (Sonic) and `OETPVault` (Plume). `OETPVault` is being shut down and will be removed from the repo after all funds are withdrawn.

Vault logic is split across two implementation contracts: `VaultCore` (user-facing mint/redeem) and `VaultAdmin` (governance functions).
`VaultCore` inherits from `VaultAdmin` and is now deployed as a single implementation contract for simplicity.
Previously they were deployed as separate implementations with a shared proxy. This was because the contract was too big to deploy as a single implementation, but after the simplification, it can now be deployed as one.

### Strategies (Yield Generation)
Located in `contracts/strategies/`. Each strategy:
- Inherits from `InitializableAbstractStrategy`
- Implements `deposit()`, `withdraw()`, `withdrawAll()`, `checkBalance()`, `collectRewardTokens()`
- Is registered with a vault and allocated collateral

Key strategies: Aave, Compound, Convex/Curve, Balancer, Morpho, Native Staking (SSV validators).
- For SSV Cluster migrations to ETH billing, use the SSV ETH payment calculator: https://ssv-eth-forecasting.vercel.app/

### OTokens
`contracts/token/OUSD.sol` and `contracts/token/OETH.sol` - rebasing ERC-20 tokens. OUSD rebases to all holders; OETH uses a similar mechanism for ETH-denominated yield.

### Oracle System
`contracts/oracle/` - price feed aggregation. `OracleRouter` routes price queries to appropriate Chainlink feeds or Curve pool oracles, with staleness checks. Each network has its own router.

### Harvesters
`contracts/harvest/` - collect reward tokens from strategies and swap to yield-bearing assets. `Harvester` for OUSD, `OETHHarvester` for OETH, network-specific variants exist.

### Automation (Talos)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [OriginProtocol/origin-dollar](https://github.com/OriginProtocol/origin-dollar) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
