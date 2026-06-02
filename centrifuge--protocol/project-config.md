---
trigger: always_on
description: Centrifuge V3 is a DeFi RWA protocol implementing ERC7540 vaults with async/sync investment logic. Modular hub-and-spoke architecture for multi-chain tokenization with automated management capabilities.
---

# CLAUDE.md

## Project Overview
Centrifuge V3 is a DeFi RWA protocol implementing ERC7540 vaults with async/sync investment logic. Modular hub-and-spoke architecture for multi-chain tokenization with automated management capabilities.

Build and test using Foundry Forge.

### Basic Commands
```bash
forge build          # Compile contracts
forge test           # Run all tests
forge snapshot       # Create gas usage snapshots
forge coverage       # Generate coverage report
forge fmt            # Auto-format Solidity code
```

### Debugging Commands
```bash
forge test -vvv                             # Test with execution traces
forge test --match-test <test_name> -vvvv  # Debug specific test with stack traces
forge debug <test_name>                     # Interactive debugger
cast call <contract> <function> <args>      # Query contract state
cast logs --address <contract>              # Analyze emitted events
cast storage <contract> <slot>              # Inspect storage slots
```

## Hub-Spoke Architecture

### Deployment Patterns
- **Cross-chain**: Hub on Ethereum, Spokes on target chains (Base, Arbitrum, etc.)
- **Same-chain**: Both Hub and Spoke on same chain (e.g., Plume)
- **Testing assumption**: Assume hub and spoke are on same chain unless specified otherwise

### Directory Structure
```
src/
├── core/                    # Core protocol module
│   ├── hub/                # Hub-side contracts
│   │   ├── Hub.sol         # Main hub logic
│   │   ├── HubHandler.sol  # Message handling
│   │   ├── HubRegistry.sol # Pool/asset registry
│   │   ├── Accounting.sol  # Investment accounting
│   │   ├── Holdings.sol    # Asset holdings tracker
│   │   ├── ShareClassManager.sol # Share class logic
│   │   └── interfaces/
│   ├── spoke/              # Spoke-side contracts
│   │   ├── Spoke.sol       # Simplified spoke logic
│   │   ├── VaultRegistry.sol # Vault registration
│   │   ├── BalanceSheet.sol # Balance tracking
│   │   ├── ShareToken.sol  # ERC20 share tokens
│   │   ├── PoolEscrow.sol  # Pool-specific escrow
│   │   ├── factories/      # Token & escrow factories
│   │   └── interfaces/
│   ├── messaging/          # Message infrastructure
│   │   ├── Gateway.sol     # Cross-chain message routing
│   │   ├── MultiAdapter.sol # Multi-protocol messaging
│   │   ├── MessageProcessor.sol # Process messages
│   │   ├── MessageDispatcher.sol # Dispatch messages
│   │   ├── GasService.sol  # Gas management
│   │   └── libraries/
│   │       └── MessageLib.sol
│   ├── libraries/
│   │   └── PricingLib.sol  # Pricing calculations
│   └── utils/
│       ├── BatchedMulticall.sol
│       └── ContractUpdater.sol # Contract update handler
├── admin/                  # Admin & governance
│   ├── Root.sol           # Root authority
│   ├── OpsGuardian.sol    # Operational guardian
│   ├── ProtocolGuardian.sol # Protocol guardian
│   ├── TokenRecoverer.sol # Token recovery
│   └── interfaces/
├── managers/              # Automation managers
│   ├── hub/
│   │   ├── NAVManager.sol # NAV automation
│   │   └── SimplePriceManager.sol # Price automation
│   └── spoke/
│       └── QueueManager.sol # Queue automation
├── vaults/                # Vault implementations
│   ├── BatchRequestManager.sol # Batch request handling
│   ├── AsyncRequestManager.sol # Async requests
│   ├── AsyncVault.sol     # ERC-7540 async vault
│   ├── SyncDepositVault.sol # Sync deposits
│   ├── SyncManager.sol    # Sync operations
│   ├── VaultRouter.sol    # Vault routing
│   ├── BaseVaults.sol     # Base implementations
│   └── factories/
├── hooks/                 # Transfer restrictions
│   ├── BaseTransferHook.sol # Base hook logic
│   ├── FreelyTransferable.sol
│   ├── FreezeOnly.sol
│   ├── FullRestrictions.sol
│   └── RedemptionRestrictions.sol
├── valuations/            # Asset valuations
│   ├── OracleValuation.sol # Oracle-based pricing
│   └── IdentityValuation.sol
├── adapters/              # Cross-chain adapters
│   ├── AxelarAdapter.sol
│   ├── ChainlinkAdapter.sol
│   ├── LayerZeroAdapter.sol
│   ├── RecoveryAdapter.sol
│   └── WormholeAdapter.sol
├── utils/                  # Utilities
│   ├── RefundEscrow.sol   # Refund handling
│   ├── RefundEscrowFactory.sol
│   └── SubsidyManager.sol
├── spell/                  # Governance spells
│   └── V2CleaningsSpell.sol
└── misc/                  # Utilities & types
    ├── Auth.sol          # Auth mixin
    ├── ERC20.sol         # Token standard
    ├── Escrow.sol        # Escrow logic
    ├── types/            # Custom types
    ├── libraries/        # Utility libraries
    └── interfaces/       # Standard interfaces

test/                        # Tests mirror src/ structure
├── core/                 # Hub & spoke tests (unit + integration)
├── vaults/               # Vault tests (unit + integration)
├── managers/             # Manager contract tests
├── hooks/                # Transfer hook tests
├── adapters/             # Cross-chain adapter tests
├── integration/          # Cross-module integration & fork tests & spell tests
└── misc/                 # Utility & library tests

script/
├── deploy/              # Deployment scripts

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [centrifuge/protocol](https://github.com/centrifuge/protocol) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-02 -->
