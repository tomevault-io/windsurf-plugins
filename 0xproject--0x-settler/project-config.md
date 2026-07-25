---
trigger: always_on
description: 0x Settler is a gas-optimized DEX aggregator settlement system that executes token swaps without holding passive allowances. It leverages [Permit2](https://github.com/Uniswap/permit2) for secure, one-time token transfers and supports multiple execution modes:
---

# 0x Settler - Development Guide

## Business Context

0x Settler is a gas-optimized DEX aggregator settlement system that executes token swaps without holding passive allowances. It leverages [Permit2](https://github.com/Uniswap/permit2) for secure, one-time token transfers and supports multiple execution modes:

- **Taker-Submitted (tokenId=2)**: Direct user transactions
- **MetaTxn (tokenId=3)**: Gasless/relayed transactions where users sign over actions
- **Intent (tokenId=4)**: Solver-authorized execution with user-signed slippage constraints
- **Bridge Settler (tokenId=5)**: Cross-chain swap execution

Key addresses:
- Deployer/Registry: `0x00000000000004533Fe15556B1E086BB1A72cEae`
- Permit2: `0x000000000022D473030F116dDEE9F6B43aC78BA3`
- AllowanceHolder (Cancun): `0x0000000000001fF3684f28c67538d4D072C22734`
- CrossChainReceiverFactory: `0x00000000000000304861c3aDfb80dd5ebeC96325`

## Architecture Overview

### Three-Flavor Settlement Pattern

```
SettlerSwapAbstract (virtual dispatch interface)
    |
    +-- SettlerBase (RFQ + UniV3 + UniV2 + Velodrome + Basic)
    |       |
    |       +-- Settler (TakerSubmitted, tokenId=2)
    |       +-- SettlerMetaTxn (tokenId=3)
    |               |
    |               +-- SettlerIntent (tokenId=4)
    |
    +-- BridgeSettlerBase (tokenId=5)
```

Each chain has its own `Common.sol` mixin that inherits from `SettlerBase` and adds chain-specific DEX support (e.g., `MainnetMixin` adds MakerPSM, MaverickV2, DodoV1/V2, UniswapV4, BalancerV3, Ekubo, EulerSwap).

### Directory Structure

```
src/
├── Settler.sol              # TakerSubmitted base
├── SettlerMetaTxn.sol       # MetaTxn base
├── SettlerIntent.sol        # Intent base
├── SettlerBase.sol          # Common settlement logic + CalldataDecoder
├── SettlerAbstract.sol      # Virtual dispatch interface
├── ISettlerActions.sol      # Action selector definitions
│
├── chains/                  # Chain-specific implementations (~27 chains)
│   └── <ChainName>/
│       ├── Common.sol       # Chain mixin (DEX integrations)
│       ├── TakerSubmitted.sol
│       ├── MetaTxn.sol
│       ├── Intent.sol
│       └── BridgeSettler.sol
│
├── core/                    # Action implementations (mixins)
│   ├── Basic.sol            # Generic pool interactions
│   ├── RfqOrderSettlement.sol
│   ├── UniswapV3Fork.sol    # V3 + 30+ forks
│   ├── UniswapV2.sol
│   ├── UniswapV4.sol
│   ├── Velodrome.sol
│   ├── MakerPSM.sol
│   ├── MaverickV2.sol
│   ├── DodoV1.sol, DodoV2.sol
│   ├── BalancerV3.sol
│   ├── Ekubo.sol
│   ├── EulerSwap.sol
│   ├── Permit2Payment.sol   # Transient storage + Permit2 integration
│   ├── SettlerErrors.sol    # Custom errors
│   └── univ3forks/          # UniV3 fork configurations
│
├── allowanceholder/         # AllowanceHolder integration
├── bridge/                  # Cross-chain bridge support
├── deployer/                # Deployment infrastructure
├── multicall/               # ERC-2771 multicall forwarding
├── utils/                   # Utilities (512Math, UnsafeMath, etc.)
└── vendor/                  # Vendored libraries (SafeTransferLib, FullMath)

test/
├── integration/             # Fork tests (run with FOUNDRY_PROFILE=integration)
├── unit/                    # Unit tests
└── utils/                   # Test utilities (Permit2Signature, ActionDataBuilder)
```

### Key Design Patterns

#### 1. Action Dispatch System

Actions are identified by 4-byte selectors from `ISettlerActions`. Dispatch happens at two levels:
- **VIP dispatch** (first action only): Direct Permit2 transfers (`TRANSFER_FROM`, `UNISWAPV3_VIP`, etc.)
- **Regular dispatch** (all actions): Pool interactions using settler-held balances

```solidity
// In Settler.execute()
if (!_dispatchVIP(action, data)) {
    if (!_dispatch(0, action, data)) {
        revertActionInvalid(0, action, data);
    }
}
```

#### 2. Transient Storage for Reentrancy

Uses EIP-1153 transient storage (`tload`/`tstore`) for:
- `_OPERATOR_SLOT`: Active operator + callback selector + callback function pointer
- `_WITNESS_SLOT`: EIP-712 witness hash for metatxns
- `_PAYER_SLOT`: Current payer (implicit reentrancy guard)

#### 3. CalldataDecoder (Lax Decoding)

Custom ABI decoder in `SettlerBase.sol` that:
- Omits bounds/overflow checking for gas efficiency
- Allows negative offsets and calldata aliasing
- Enables advanced calldata reuse patterns

#### 4. Mixin-Based Composition

Chain-specific functionality is composed via mixins. When adding a new DEX:
1. Create action implementation in `src/core/`
2. Add to chain's `Common.sol` mixin
3. Add action selector to `ISettlerActions.sol`
4. Implement in `_dispatch()` and optionally `_dispatchVIP()`

## Solidity Contribution Guidelines

### General Principles

- **Think first, code second**: Minimize lines changed; consider ripple effects
- **Prefer simplicity**: Fewer moving parts = fewer bugs and lower audit overhead
- **Contract size matters**: This codebase is at the edge of the 24KB limit

### Assembly Usage

| Rule | Rationale |
|------|-----------|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [0xProject/0x-settler](https://github.com/0xProject/0x-settler) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
