---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in bootstrapper-v2.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in bootstrapper-v2.

## Project Overview

Bootstrapper-v2 is a CLI application for bootstrapping complete Madara networks with Ethereum
as the base layer. It uses a factory pattern for atomic, efficient contract deployments across
both L1 (Ethereum) and L2 (Madara).

**Key capabilities:**

- Two-phase deployment approach (L1 setup → L2 setup)
- Factory contracts for atomic deployments
- Automatic bridge configuration between layers
- Complete cross-layer communication setup

**Version Requirement:** Requires Madara node with StarkNet protocol version 0.14.0 (not backwards compatible)

## Common Commands

### Building

```bash
cargo build --release
```

### Running

**Important:** `setup-base` MUST run before `setup-madara`. Two separate executions are required.

```bash
# Setup Base Layer (L1)
RUST_LOG=debug cargo run --bin bootstrapper-v2 -- \
  setup-base --config-path configs/config.json \
  --addresses-output-path output/addresses.json

# Setup Madara (L2) - requires base layer addresses from previous step
RUST_LOG=debug cargo run --bin bootstrapper-v2 -- \
  setup-madara --config-path configs/config.json \
  --base-addresses-path output/addresses.json \
  --output-path output/madara_addresses.json
```

## Architecture Overview

### CLI Commands

| Command        | Purpose                                                   |
| -------------- | --------------------------------------------------------- |
| `setup-base`   | Deploy L1 infrastructure (Factory, CoreContract, bridges) |
| `setup-madara` | Deploy L2 infrastructure (UDC, MadaraFactory, bridges)    |

### Module Structure

**`cli/`** - Command-line interface

- `mod.rs`: CLI argument structure using `clap`
- `setup_base.rs`: Base layer setup command parameters
- `setup_madara.rs`: Madara setup command parameters

**`config.rs`** - Configuration management

- `BaseConfigOuter`: Top-level config for base layer
- `MadaraConfigOuter`: Top-level config for Madara
- `BaseLayerConfig`: Supports both Ethereum and Starknet base layers

**`setup/base_layer/`** - L1 setup logic

- `mod.rs`: `BaseLayerSetupTrait` definition
- `ethereum/mod.rs`: `EthereumSetup` implementation
- `ethereum/factory.rs`: Factory contract deployment and setup
- `ethereum/implementation_contracts.rs`: Contract artifact mappings
- `ethereum/constants.rs`: Artifact paths
- `ethereum/config_hash.rs`: Dynamic config hash computation with DA public keys support
- `ethereum/error.rs`: `EthereumError` enum with specialized error types
- `starknet.rs`: Placeholder for Starknet-as-base-layer support

**`setup/madara/`** - L2 setup logic

- `mod.rs`: `MadaraSetup` implementation
- `bootstrap_account.rs`: Bootstrap account for initial declaration
- `class_contracts.rs`: Cairo contract class definitions
- `constants.rs`: Artifact paths for Cairo contracts

**`error/`** - Error handling

- `mod.rs`: Main `BootstrapperError` enum
- `madara.rs`: Madara-specific errors

**`utils.rs`** - Shared utilities

- Transaction waiting and receipt handling
- Contract declaration
- Address extraction from events

### Deployment Flow

**Base Layer (L1 - Ethereum):**

```text
1. Deploy implementation contracts (CoreContract, Manager, Registry, MultiBridge, EthBridge, EthBridgeEIC)
2. Deploy Factory contract with implementation references
3. Call Factory.setup() → BaseLayerContractsDeployed event
4. Extract addresses: CoreContract, Manager, Registry, MultiBridge, EthBridge
5. Post-Madara
```

**Madara (L2 - StarkNet):**

```text
1. Bootstrap account declare (OpenZeppelin Account with special nonce=0)
2. Deploy user account via OpenZeppelin AccountFactory
3. Declare Cairo contracts (TokenBridge, ERC20, EIC, UniversalDeployer, MadaraFactory)
4. Deploy UniversalDeployer (UDC)
5. Deploy MadaraFactory with class hashes and L1 bridge addresses
6. Call MadaraFactory.deploy_bridges()
7. Extract addresses: L2 ETH Token, L2 ETH Bridge, L2 Token Bridge, L2 Fee Token
```

**Post-Madara Setup (L1 finalization):**

```text
1. Call setL2Bridge() on L1 bridges to set L2 bridge addresses
2. Call enrollTokenBridge() to register token bridge
3. Poll L2 for enrolled token (300s timeout, 5s interval)
4. Validate fee token matches configured value
5. Update config hash with actual deployed fee token address
```

## Configuration

### Config File Format (`configs/config.json`)

```json
{
  "base_layer": {
    "layer": "ETHEREUM",
    "rpc_url": "http://localhost:8545",
    "implementation_addresses": {},
    "deploy_test_contracts": true,
    "l1_token_address": "0x...",
    "core_contract_init_data": {
      "programHash": "0x1",
      "aggregatorProgramHash": "0x0",
      "verifier": "0x0...",
      "configHash": "0x0",
      "state": {
        "globalRoot": "0x0",
        "blockNumber": "0x0",
        "blockHash": "0x0"
      }
    },
    "config_hash_config": {
      "version": "StarknetOsConfig3",
      "madara_chain_id": "MADARA_DEVNET",
      "madara_fee_token": "0x...",
      "da_public_keys": []
    }
  },
  "madara": {
    "rpc_url": "http://localhost:9945"
  }
}
```

**Config fields:**

- `deploy_test_contracts`: If true, deploys mock L1 token for testing (default: false)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [madara-alliance/madara](https://github.com/madara-alliance/madara) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
