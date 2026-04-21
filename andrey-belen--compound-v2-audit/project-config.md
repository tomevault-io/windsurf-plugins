---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a comprehensive security audit environment for Compound V2 protocol analysis, designed for educational and defensive security purposes. The project includes:

- **Security Testing Framework**: Complete test suite for identifying DeFi vulnerabilities
- **Attack Vector Simulations**: Flash loan, reentrancy, oracle manipulation, and liquidation attacks
- **Historical Bug Analysis**: Recreation of known Compound V2 vulnerabilities for learning
- **Audit Infrastructure**: Logging, monitoring, and reporting tools for security analysis

## Architecture

### Core Components

- `src/AuditBase.sol` - Base contract with common utilities (logging, time manipulation, token dealing, address constants)
- `src/CompoundHelpers.sol` - Helper functions for Compound V2 interactions (supply, borrow, liquidate) with interfaces
- `src/OracleManipulator.sol` - Price oracle manipulation utilities for testing vulnerabilities with DEX interactions
- `src/Counter.sol` - Simple counter contract (template/example)

### Contract Inheritance Pattern
All test contracts follow this pattern:
```solidity
contract MyAuditTest is Test, AuditBase, CompoundHelpers, OracleManipulator {
    // Inherits Foundry Test + audit utilities + Compound helpers + oracle manipulation
}
```

### Test Structure

```
test/
├── exploits/               # Attack vector test suites
│   ├── FlashLoanAttacks.t.sol
│   ├── ReentrancyTests.t.sol
│   └── CompoundHistoricalBugs.t.sol
├── unit/                   # Unit tests
├── integration/            # Integration tests
└── fixtures/               # Test data and configuration
```

### Key Directories

- `audit-reports/` - Generated audit reports and findings
- `scripts/` - Deployment and utility scripts (setup-env.sh, DeployAuditEnvironment.s.sol)
- `lib/` - External dependencies (OpenZeppelin, Compound protocol, forge-std)
- `out/` - Compiled contract artifacts and build information
- `cache/` - Foundry compilation cache

### Pre-configured Constants
The `AuditBase.sol` provides pre-configured mainnet addresses:

**Tokens**: USDC, WETH, DAI, USDT, COMP_TOKEN  
**Compound V2**: COMPTROLLER, CUSDC, CDAI, CETH, CUSDT, PRICE_ORACLE  
**Test Accounts**: ADMIN, USER1, USER2, LIQUIDATOR, ATTACKER, WHALE

## Development Commands

### Environment Setup
```bash
# Initial setup (run once) - sets up .env, git hooks, dependencies
./scripts/setup-env.sh

# Manual setup alternative
cp .env.example .env
# Edit .env with your RPC URLs and API keys
source .env

# Install dependencies and build
forge install
forge build
```

### Core Testing Commands
```bash
# Run all tests with mainnet fork
forge test --fork-url $ETH_RPC_URL

# Run specific vulnerability test contracts
forge test --match-contract FlashLoanAttacksTest --fork-url $ETH_RPC_URL -vv
forge test --match-contract ReentrancyTestsTest --fork-url $ETH_RPC_URL -vv
forge test --match-contract CompoundHistoricalBugsTest --fork-url $ETH_RPC_URL -vv

# Run individual test files
forge test --match-path test/exploits/FlashLoanAttacks.t.sol --fork-url $ETH_RPC_URL -vv
forge test --match-path test/exploits/ReentrancyTests.t.sol --fork-url $ETH_RPC_URL -vv

# Run specific test functions
forge test --match-test test_FlashLoanPriceManipulation --fork-url $ETH_RPC_URL -vvv
forge test --match-test test_SupplyReentrancy --fork-url $ETH_RPC_URL -vvv

# Quick security test suite (if script exists)
./scripts/test-quick.sh

# Full test suite with coverage (if script exists)
./scripts/test-full.sh

# Gas usage analysis (if script exists)
./scripts/analyze-gas.sh
```

### Build and Deploy
```bash
# Build contracts
forge build

# Deploy audit environment
forge script scripts/DeployAuditEnvironment.s.sol --rpc-url $ETH_RPC_URL

# Run with different profiles
forge test --profile ci          # CI profile (more fuzz runs)
forge test --profile intense     # Intensive testing
```

### Debugging and Analysis
```bash
# Run with maximum verbosity
forge test --fork-url $ETH_RPC_URL -vvvv

# Gas reporting
forge test --gas-report --fork-url $ETH_RPC_URL

# Coverage analysis
forge coverage --fork-url $ETH_RPC_URL --report lcov
```

## Critical Configuration

### Environment Variables Required
- `ETH_RPC_URL` - Mainnet RPC endpoint (Alchemy/Infura)
- `PRIVATE_KEY` - Private key for deployments (TEST KEY ONLY! Default: Anvil test key)
- `ETHERSCAN_API_KEY` - For contract verification (optional)
- `POLYGON_RPC_URL` - Polygon RPC endpoint (optional, for multi-chain testing)
- `ARBITRUM_RPC_URL` - Arbitrum RPC endpoint (optional, for multi-chain testing)

### Fork Configuration
- **Fork Block**: 19,000,000 (configured in foundry.toml)
- **Reason**: Block with stable Compound V2 deployment for reliable testing
- **Gas Limit**: 30M gas for complex attack simulations

### Security Settings
- All tests include safety checks to prevent mainnet operations
- Private key validation in git hooks
- Automated security scanning in pre-commit hooks

## Test Categories and Purpose

### 1. Flash Loan Attacks (`FlashLoanAttacks.t.sol`)
**Educational Purpose**: Understanding how flash loans can be weaponized

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/andrey-belen) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
