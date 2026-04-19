---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## CRITICAL: Use Type Safety
**Never use the "any" type in TypeScript files**, that defeats the purpose of using TypeScript. Type safety protects us from mistakes. Use proper types.

### Critical Lesson: Failing Tests Reveal Production Bugs

**Never rush to "fix" a failing test by changing test data** - investigate the root cause first. A systematic production workflow test revealed multiple division by zero vulnerabilities:

1. **Bug 1**: `getPricePerFullShare()` division by zero when `tokenBalance == 0` but `totalSupply > 0`
2. **Bug 2**: Withdrawal processing division by zero when `totalShares[tokenIdx] == 0`

**The Fix**: Added proper edge case handling and sanity checks rather than changing test numbers.

**The Lesson**: Complex integration tests that simulate real user workflows often catch edge cases that unit tests miss. These bugs could have caused catastrophic failures in production.

**Remember**: When tests fail, ask "What should the code do?" not "How can I make the test pass?"

### Critical Lesson: Tests Define Business Requirements, Not Implementation Constraints

**When your implementation reveals better business practices than what tests expect, update the TESTS to reflect the improved requirements.**

**The Principle**: Tests should enforce good business requirements. If implementation demonstrates better practices (UX, security, efficiency, maintainability), those practices should become the new business requirements, and tests should be updated accordingly.

**Common Scenarios Where This Applies**:
- **Security**: Implementation adds input validation → Update tests to expect validation
- **UX**: Implementation adds confirmation flows → Update tests to expect confirmations  
- **Gas Efficiency**: Implementation batches operations → Update tests to expect batching
- **Error Handling**: Implementation adds graceful fallbacks → Update tests to expect fallbacks
- **Performance**: Implementation adds caching → Update tests to expect cached behavior

**Key Questions When Tests Conflict with Implementation**:
1. "Does this implementation demonstrate better business practices?"
2. "Should this improvement become our standard requirement?"
3. If yes → Update tests to enforce the better pattern
4. If no → Fix implementation to meet existing requirements

## Project Overview

Arca is a multi-protocol decentralized vault system for automated liquidity provision on the Sonic blockchain. Originally built for Metropolis DLMM (Dynamic Liquidity Market Maker) pools, the system is expanding to support multiple DEX protocols including Shadow (Ramses V3) concentrated liquidity pools. The project leverages battle-tested code from audited Metropolis Maker Vaults as its foundation, providing intelligent vault management with automated reward compounding and yield optimization through Python bot rebalancing.

## Repository Structure

This is a full-stack DeFi project with:
- **Metropolis Contracts** (`/contracts-metropolis/`) - Core vault system for DLMM pools
- **Shadow Contracts** (`/contracts-shadow/`) - Shadow/Ramses V3 integration (GPL-3.0 licensed)
- **Frontend dApp** (`/UI/`) - React-based user interface  
- **Testing Suite** (`/test/`) - Comprehensive test coverage
- **Deployment System** (`/scripts/`) - TypeScript deployment infrastructure
- **External Dependencies** (`/lib/`) - Git submodules (joe-v2)
- **Archive** (`/archive/`) - Previous implementation for reference

## Core Architecture

### Multi-Protocol Support
The system supports multiple DEX protocols through a modular architecture:

**Metropolis (DLMM)**:
- `BaseVault`, `OracleVault`, `OracleRewardVault` - Vault implementations
- `Strategy` - Manages bin-based liquidity positions
- Uses `ILBPair` interface for liquidity book pairs

**Shadow (Concentrated Liquidity)**:
- `BaseShadowVault`, `OracleShadowVault`, `OracleRewardShadowVault` - Shadow-specific vaults
- `ShadowStrategy` - Manages NFT-based concentrated liquidity positions
- Uses `IRamsesV3Pool` and `INonfungiblePositionManager` interfaces

### Factory System
- `VaultFactory` - Central factory supporting multiple vault and strategy types
- Handles deployment of protocol-specific implementations
- Manages cross-protocol configuration (NPM addresses, voters, etc.)

### Key Design Patterns
- **Upgradeable Proxy Pattern**: All contracts use OpenZeppelin's UUPS upgradeable contracts
- **Protocol-Agnostic Interfaces**: Common interfaces (`IStrategyCommon`) with protocol-specific extensions
- **Immutable Clone Pattern**: Strategies use immutable data for gas-efficient deployments
- **Queue-Based Processing**: Deposits and withdrawals are queued and processed during rebalance
- **Dual Token Shares**: Separate share tracking for TokenX and TokenY with proportional ownership
- **Position Management**: 
  - Metropolis: Direct bin management with add/remove liquidity
  - Shadow: NFT-based positions with mint/burn lifecycle

## Development Commands

### Core Development
```bash
# Compile contracts
npm run compile

# Run tests
npm run test

# Run linting
npm run lint

# Fix linting issues
npm run lint:fix
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/arcaDeFAI) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
