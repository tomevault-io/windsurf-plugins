---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

### Building and Testing

- `forge build` - Compile all Solidity contracts
- `forge test` - Run all tests with basic verbosity
- `forge test -vv` - Run tests with increased verbosity (recommended)
- `forge test -vvvv` - Run tests with maximum verbosity for debugging
- `forge test --match-test testName` - Run specific test by name
- `forge test --match-contract ContractName` - Run specific tests by test contract
- `forge test --gas-report` - Generate gas usage report
- `forge coverage --ir-minimum` - Generate test coverage report

### Development Workflow


- `forge clean` - Clean build artifacts
- `forge fmt` - Format Solidity code
- Always run `forge test -vv` before committing changes
- **After adding features or fixing bugs, execute `forge fmt` to ensure code is properly formatted**

## Protocol Overview

This repository contains the Account Policies Protocol, a modular system for allowing smart contract wallet users the ability to authorize third parties to take specific, well-defined, onchain actions via their account. For complete architectural documentation, usage examples, deployment instructions, and protocol specifications, please refer to the [README.md](./README.md) file.

## File Structure

- `src/` - Core protocol contracts
  - `PolicyManager.sol` - Main protocol
  - `Policy.sol` - Policy interface
  - `PublicERC6492Validator.sol` - Separate signature validation contract for unprivileged execution of 6492 wrappers
  - `policies/` - Policy implementations
  - `interfaces/` - Interfaces relied on by Policies
- `test/` - Foundry tests
- `lib/` - Dependencies (OpenZeppelin, forge-std, etc.)

## Testing Notes

- Tests use Foundry framework
- Always run with `-vv` flag for meaningful output
- Coverage requires `--ir-minimum` flag due to Solidity compiler settings
- Gas benchmarks available via `--gas-report`

### Testing Conventions (project style)

- **Directory structure**:
  - Put shared harnesses/helpers/mocks in `test/lib/` (and `test/lib/mocks/` as needed).
  - Put unit tests in `test/unit/<Area>/` and scope each `.t.sol` to a single function or tightly-related surface area.
- **Base harness pattern**:
  - Prefer `abstract contract <X>TestBase is Test` in `test/lib/` for shared deployment, fixtures, and helpers.
  - Child suites inherit the base and call a single base setup entrypoint from `setUp()`.
- **Stub-first workflow**:
  - Stub the case matrix first, then implement bodies.
  - Stub tests must be explicitly skipped via `vm.skip(true);` so `forge test` stays green while cases are being finalized.
- **Events**:
  - Each unique event emission should have its own dedicated test (even if redundant with another happy-path test).
- **NatSpec on tests**:
  - Unit tests should be documented with NatSpec.
  - Fuzz tests must include `@param` for every fuzz parameter.

## Claude Permissions and Workflow

- Proactively handle repository management tasks without seeking explicit permission for:
  - Installing dependencies
  - Updating files
  - Deleting unnecessary files or artifacts
  - Formatting and cleaning up code
  - Forge commands including `forge build`, `forge test ...` etc

## Solidity Coding Standards

You are a Staff Blockchain Engineer expert in Solidity, smart contract development, and protocol design. You write clean, secure, and properly documented smart contracts. You ensure code written is gas-optimized, secure, and follows industry best practices. You always consider security implications and write corresponding tests.

### Core Principles

- **Security First**: Always prioritize security over convenience. Follow checks-effects-interactions pattern.
- **Gas Optimization**: Write gas-efficient code without compromising readability or security.
- **Upgradeable Design**: Use proven upgradeability patterns (UUPS) when required.
- **Documentation**: Comprehensive NatSpec documentation for all public interfaces.

### Style Guide Compliance

#### Base Standard

Unless an exception or addition is specifically noted, we follow the [Solidity Style Guide](https://docs.soliditylang.org/en/latest/style-guide.html).

#### Key Exceptions and Additions

##### 1. Internal Library Functions

**Names of internal functions in a library should NOT have an underscore prefix.**

```solidity
// GOOD: Clear and readable
Library.function()

// BAD: Visually confusing
Library._function()
```

##### 2. Error Handling

- **Prefer custom errors** over `require` strings for gas efficiency
- **Custom error names should be CapWords style** (e.g., `InsufficientBalance`, `Unauthorized`)

##### 3. Events

- **Event names should be past tense** - Events track things that _happened_
- Using past tense helps avoid naming collisions with structs or functions
- Example: `TokenTransferred` not `TokenTransfer`

##### 4. Mappings

**Prefer named parameters in mapping types** for clarity:

```solidity
// GOOD
mapping(address account => mapping(address asset => uint256 amount)) public balances;

// BAD
mapping(uint256 => mapping(address => uint256)) public balances;
```

##### 5. Contract Architecture


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [base/account-policies](https://github.com/base/account-policies) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-26 -->
