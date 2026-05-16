---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Foundry-based Ethereum smart contract development project. Foundry is a Rust-based toolkit that provides a complete development environment for Solidity smart contracts.

- **Foundry Documentation**: https://getfoundry.sh/

### Project Documentation

- **[SPEC.md](./SPEC.md)** - Complete specification for the Collectible Casts system, including architecture, goals, and detailed contract requirements
- **[PLAN.md](./PLAN.md)** - High-level implementation plan outlining development phases and testing strategy
- **[TASKS.md](./TASKS.md)** - Detailed task breakdown with specific implementation steps and test scenarios

## Essential Commands

### Building and Testing

```bash
forge build              # Compile all contracts
forge build --sizes      # Compile and show contract sizes
forge test              # Run all tests
forge test -vvv         # Run tests with maximum verbosity (useful for debugging)
forge test --match-test testName  # Run specific test
forge test --match-contract ContractName  # Run tests for specific contract
forge test --gas-report  # Run tests with gas usage report

# Fuzz testing profiles
forge test                    # Default: 2048 fuzz runs
FOUNDRY_PROFILE=ci forge test # CI: 10,000 fuzz runs
FOUNDRY_PROFILE=deep forge test # Deep: 50,000 fuzz runs
forge test --fuzz-runs 100000 --fuzz-seed 0x123  # Custom deterministic deep fuzzing
```

### Code Quality

```bash
forge fmt               # Format all Solidity files (120 char line length)
forge fmt --check       # Check formatting without modifying files
forge snapshot          # Generate gas usage snapshots
forge snapshot --check  # Compare gas usage against .gas-snapshot
forge coverage          # Generate test coverage report (minimum 94%)
python3 script/check-coverage.py  # Verify 100% coverage for production contracts
```

### Local Development

```bash
anvil                   # Start local Ethereum node for testing
anvil --fork-url <RPC_URL>  # Fork mainnet for testing
```

### Deployment

```bash
forge script script/Counter.s.sol:CounterScript --rpc-url <RPC_URL> --private-key <PRIVATE_KEY> --broadcast
forge script script/Counter.s.sol:CounterScript --rpc-url <RPC_URL> --private-key <PRIVATE_KEY> --broadcast --verify
```

## CRITICAL: Test-Driven Development (TDD) Process

**This project STRICTLY follows TDD methodology. No production code should be written without a failing test first.**

### TDD Workflow - RED → GREEN → REFACTOR → COMMIT

1. **RED**: Write a failing test first

   ```bash
   # Write test in test/ContractName.t.sol
   forge test --match-test test_NewFeature -vvv  # See it fail
   ```

2. **GREEN**: Write minimal code to pass

   ```bash
   # Write just enough code in src/ContractName.sol
   forge test --match-test test_NewFeature  # See it pass
   ```

3. **REFACTOR**: Improve code while keeping tests green

   ```bash
   # Refactor implementation
   forge test  # Ensure all tests still pass
   forge fmt   # Format code
   ```

4. **COMMIT**: Only commit when all tests pass
   ```bash
   forge test            # Final test run
   forge coverage        # Check coverage
   python3 script/check-coverage.py  # Verify 100% coverage
   git add -A
   git commit -m "feat: implement feature with tests"
   ```

### TDD Rules

- **NEVER write production code without a failing test**
- **NEVER write more production code than needed to pass the test**
- **NEVER refactor with failing tests**
- **ALWAYS run tests before committing**
- **Target 100% test coverage** - use `forge coverage` to verify
- **Tests are first-class code** - apply the same quality standards to tests as production code
- **Prefer fuzz tests** - write fuzz tests for all functions that accept parameters

## Coding Standards and Conventions

### File Structure and Code Organization

- All Solidity files must include:
  ```solidity
  // SPDX-License-Identifier: UNLICENSED
  pragma solidity ^0.8.30;
  ```
- Import order (separated by blank lines):
  1. External dependencies (OpenZeppelin, forge-std)
  2. Interfaces
  3. Abstract contracts/libraries
  4. Relative imports
- Example:

  ```solidity
  import {ERC721} from "openzeppelin/contracts/token/ERC721/ERC721.sol";

  import {IIdRegistry} from "./interfaces/IIdRegistry.sol";

  import {Signatures} from "./abstract/Signatures.sol";
  ```

- Contract internal organization:
  1. Custom errors
  2. Events
  3. Constants
  4. State variables (grouped by visibility)
  5. Constructor
  6. Functions (external → public → internal → private)

### Naming Conventions

- **Contract files**: PascalCase (e.g., `TokenVault.sol`, `AccessManager.sol`)
- **Test files**: `ContractName.t.sol` (e.g., `TokenVault.t.sol`)
- **Script files**: `ContractName.s.sol` or descriptive names (e.g., `Deploy.s.sol`)
- **Test contracts**: `ContractNameTest` (e.g., `TokenVaultTest`)
- **Test functions**:
  - `test_FunctionName()` for unit tests
  - `testFuzz_FunctionName(uint256 param)` for fuzz tests
  - `testFail_FunctionName()` for tests expecting reverts
  - Use underscores in test function names for readability

### Error Handling


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [farcasterxyz/collectible-casts](https://github.com/farcasterxyz/collectible-casts) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
