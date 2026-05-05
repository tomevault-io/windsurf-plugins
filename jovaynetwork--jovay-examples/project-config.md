---
trigger: always_on
description: This document provides comprehensive guidance for AI agents working with the jovay-examples repository. It covers repository structure, workflows, conventions, and best practices.
---

# AGENTS.md - AI Agent Guide for Jovay Examples

This document provides comprehensive guidance for AI agents working with the jovay-examples repository. It covers repository structure, workflows, conventions, and best practices.

## Table of Contents

1. [Repository Overview](#repository-overview)
2. [Examples Registry System](#examples-registry-system)
3. [Adding New Examples](#adding-new-examples)
4. [Modifying Existing Examples](#modifying-existing-examples)
5. [CI/CD Integration](#cicd-integration)
6. [Code Standards](#code-standards)
7. [Common Tasks](#common-tasks)
8. [Related Documentation](#related-documentation)

## Repository Overview

### Purpose

The jovay-examples repository hosts small, focused, runnable examples that help developers build on **Jovay Network**. Each example is self-contained and includes its own README with prerequisites and step-by-step instructions.

### Key Principles

- **Self-contained**: Each example should work independently
- **Minimal dependencies**: Keep examples focused and easy to reproduce
- **Well-documented**: Every example must have a clear README
- **Tested**: Examples should include tests when feasible
- **No secrets**: Never commit private keys, API keys, or sensitive configuration

### Repository Structure

```
.
├── chainlink_examples/          # Chainlink-related examples
│   └── ccip_example/            # Example: CCIP messaging + token transfer
├── ci/                          # CI/CD scripts
│   ├── list_examples.py         # Parses examples.yaml, generates CI matrix
│   ├── run_example.sh           # Main CI runner (dispatches by type)
│   └── solidity_foundry.sh      # Foundry-specific checks
├── foundry_examples/            # Foundry tutorial examples
│   ├── token_example/           # ERC-20 token example
│   ├── nft_example/             # ERC-721 NFT example
│   └── staking_example/         # Staking contract example
├── hardhat_examples/            # Hardhat tutorial examples
│   ├── token_example/           # ERC-20 token example
│   ├── nft_example/             # ERC-721 NFT example
│   └── staking_example/         # Staking contract example
├── khalani_examples/            # Khalani intent market examples
│   └── cross_chain_swap/        # Cross-chain swap dApp (Vite + React)
├── .github/workflows/           # GitHub Actions workflows
│   └── examples.yml             # CI workflow definition
├── examples.yaml                # Single source of truth for examples registry
├── README.md                    # Main repository README
├── CONTRIBUTING.md              # Contribution guidelines
└── LICENSE                      # MIT License
```

### Key Files

- **`examples.yaml`**: Registry of all examples with metadata and test configuration
- **`ci/list_examples.py`**: Validates and parses `examples.yaml`, outputs JSON or GitHub Actions matrix
- **`ci/run_example.sh`**: Dispatches to type-specific test runners
- **`ci/solidity_foundry.sh`**: Runs Foundry checks (fmt, build, test)

## Examples Registry System

### Overview

The `examples.yaml` file is the **single source of truth** for all examples in the repository. It defines:

- Example paths (relative to repo root)
- Example types (e.g., `solidity`)
- Descriptions
- Test configurations

### Schema

```yaml
version: 1

examples:
  - path: chainlink_examples/ccip_example
    type: solidity
    description: "Chainlink CCIP example (Foundry): messaging + token transfers between Sepolia and Jovay Testnet."
    test:
      solidity:
        foundry:
          fmt: true          # Run forge fmt --check
          build: true        # Run forge build
          test:              # Run forge test
            offline: true    # Use --offline flag for tests

  - path: khalani_examples/cross_chain_swap
    type: frontend
    description: "Khalani cross-chain swap example dApp (Vite + React): swap ETH/USDC between Ethereum Mainnet and Jovay Network using Intent Markets."
```

### Validation Rules

The `ci/list_examples.py` script validates:

1. **Version**: Must be `1`
2. **Examples list**: Must be non-empty
3. **Path uniqueness**: Each path must be unique
4. **Path existence**: Each path must exist as a directory
5. **Required fields**: `path`, `type`, `description` must be non-empty strings

### Adding to Registry

When adding a new example:

1. Create the example directory and files
2. Add an entry to `examples.yaml` with:
   - Unique `path` (relative to repo root)
   - `type` (`solidity` or `frontend`)
   - Clear `description`
   - Appropriate `test` configuration (for `solidity` type)

### CI Integration

The GitHub Actions workflow (`.github/workflows/examples.yml`):

1. Reads `examples.yaml` via `ci/list_examples.py`
2. Generates a matrix of examples
3. Runs checks for each example in parallel
4. For `solidity` type, executes `ci/solidity_foundry.sh`
5. For `frontend` type, performs basic file structure validation

## Adding New Examples

### Step-by-Step Process

1. **Create Example Directory**
   ```
   mkdir -p <category>_examples/<example_name>
   ```

2. **Create Example Files**
   - Source code in `src/` (for Foundry projects)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jovaynetwork/jovay-examples](https://github.com/jovaynetwork/jovay-examples) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
