---
trigger: always_on
description: NFT Marketplace - A comprehensive Solidity-based marketplace for creating, trading, auctioning, and staking NFTs.
---

# Project Context

NFT Marketplace - A comprehensive Solidity-based marketplace for creating, trading, auctioning, and staking NFTs.

**Tech Stack:**
- Solidity `^0.8.24`
- Foundry (Forge, Anvil, Cast)
- OpenZeppelin Contracts & Contracts Upgradeable
- Chainlink VRF v2.5
- pnpm workspace (monorepo)

**Core Modules:**
- **Collection Factory**: Create ERC721 collections via beacon proxies
- **Marketplace**: EIP-712 signed orders/offers for trading
- **Auctions**: Time-based bidding system
- **Staking**: Stake NFTs for ETH rewards
- **Governance**: MultisigTimelock for admin operations

---

## Commands

```bash
# Build contracts
cd foundry && forge build

# Run all tests
cd foundry && forge test

# Run specific test file
cd foundry && forge test --match-path test/unit/marketplace/*.sol

# Run specific test function
cd foundry && forge test --match-test test_ExecuteOrder_Succeeds

# Run with verbosity (show logs, traces)
cd foundry && forge test -vvv

# Run with gas reporting
cd foundry && forge test --gas-report

# Format code
cd foundry && forge fmt

# Local deployment
anvil  # terminal 1
cd foundry && forge script script/DeployFactory.s.sol --rpc-url http://localhost:8545 --broadcast  # terminal 2

# Generate coverage report
cd foundry && forge coverage

# Frontend (not yet implemented)
cd frontend && pnpm dev
```

---

## Architecture Overview

### Directory Structure

```
nft-marketplace/
├── foundry/                    # Smart contracts (main codebase)
│   ├── src/                    # Contract implementations
│   │   ├── ERC721Collection.sol       # NFT collection (upgradeable, VRF, royalties)
│   │   ├── Factory.sol                # Creates collections via beacon proxy
│   │   ├── Marketplace.sol            # Orders & offers with EIP-712 signatures
│   │   ├── Auction.sol                # Time-based auctions
│   │   ├── Staking.sol                # NFT staking rewards
│   │   ├── MultisigTimelock.sol       # Governance (multisig + timelock)
│   │   ├── VRFAdapter.sol             # Chainlink VRF v2.5 integration
│   │   ├── ERC721CollectionBeacon.sol # Beacon for collection proxies
│   │   ├── SwapAdapter.sol            # ETH/WETH wrapper
│   │   └── interfaces/
│   │       ├── IFactory.sol
│   │       └── IERC721Collection.sol
│   ├── test/
│   │   ├── Base.t.sol                 # Base test contract with constants
│   │   ├── unit/                      # Unit tests by module
│   │   │   ├── auction/
│   │   │   ├── erc721collection/
│   │   │   ├── factory/
│   │   │   ├── marketplace/
│   │   │   └── staking/
│   │   └── helpers/                   # Test utilities & mocks
│   │       ├── DeployHelpers.s.sol
│   │       ├── MarketplaceTestHelper.sol
│   │       ├── FactoryHelper.sol
│   │       ├── AuctionTestBase.sol
│   │       ├── StakingTestHelper.sol
│   │       └── Mocks.sol
│   ├── script/
│   │   └── DeployFactory.s.sol
│   ├── lib/                           # Dependencies (OZ, Chainlink, forge-std)
│   ├── foundry.toml
│   └── remappings.txt
├── frontend/                   # React + Vite (skeleton only)
├── CLAUDE.md
└── pnpm-workspace.yaml
```

### Proxy Architecture

**UUPS Proxies** (upgradeable single-instance contracts):
- Factory, Marketplace, Auction, Staking, VRFAdapter, MultisigTimelock

**Beacon Proxies** (multiple instances sharing one implementation):
- All ERC721Collection instances created via Factory

### Contract Dependencies

```
MultisigTimelock
       ↓ (controls)
    Factory ←────────────────┐
       ↓ (creates)           │
ERC721CollectionBeacon       │
       ↓ (points to)         │
ERC721Collection (impl)      │
       ↓ (instances use)     │
BeaconProxy                  │
       ↓ (trades on)         │
  Marketplace ───────────────┤
       ↓                     │
   Auction ──────────────────┤
       ↓                     │
  StakingNFT ────────────────┘
```

### Key Constants

| Constant | Value | Description |
|----------|-------|-------------|
| MAX_SUPPLY | 20,000 | Max tokens per collection |
| MAX_ROYALTY | 1000 (10%) | Max royalty in basis points |
| MAX_MARKETPLACE_FEE | 500 (5%) | Max marketplace fee |
| MAX_AUCTION_FEE | 500 (5%) | Max auction fee |
| MIN_DURATION | 30 minutes | Min auction duration |
| MAX_DURATION | 7 days | Max auction duration |
| MIN_PRICE | 1000 wei | Minimum price allowed |
| FEE_DENOMINATOR | 10,000 | Basis points divisor |

---

## Key Patterns

### Writing New Contracts

**UUPS Upgradeable Contract Template:**
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.24;

import {Initializable} from "openzeppelin-contracts-upgradeable/proxy/utils/Initializable.sol";
import {UUPSUpgradeable} from "openzeppelin-contracts-upgradeable/proxy/utils/UUPSUpgradeable.sol";
import {ReentrancyGuard} from "openzeppelin-contracts/utils/ReentrancyGuard.sol";

interface IMultisigTimelock {
    function verifyCurrentTransaction() external view;
}

contract NewContract is Initializable, UUPSUpgradeable, ReentrancyGuard {
    // ========== STATE VARIABLES ==========
    address private _multisigTimelock;

    // ========== ERRORS ==========
    error NotAMultisigTimelock();
    error ZeroAddress();

    // ========== MODIFIERS ==========
    modifier onlyMultisig() {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Gravikko) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
