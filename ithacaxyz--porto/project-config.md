---
trigger: always_on
description: This file provides essential information for Claude AI (and other AI assistants) working with the Porto codebase.
---

# CLAUDE.md - AI Assistant Guide for Porto

This file provides essential information for Claude AI (and other AI assistants) working with the Porto codebase.

## Mandatory GitHub Interaction Rules

### Required Tools

1. **GitHub MCP**: `mcp__github__*` functions (authenticated, no rate limits)
2. **GitHub CLI**: `gh` commands (authenticated via CLI)
3. **NEVER**: Direct API calls, web scraping, or unauthenticated requests

### Examples

✅ **Correct**: `mcp__github__get_pull_request` or `gh pr view`
❌ **Wrong**: Direct API calls to `api.github.com`

### Pull Requests

- You are allowed to open pull requests for this repository.
- Always use `mcp__github__create_pull_request` or `gh pr create` to create pull requests.

## About Porto

Porto is a Next-Generation Account Stack for Ethereum that provides a secure, scalable, and user-friendly account system. It's built as a TypeScript monorepo with multiple integrations and applications.

## Helpful Links

- [Porto Docs](https://porto.sh)
- [Porto `llms.txt`](https://porto.sh/llms.txt)
- [Porto `llms-full.txt`](https://porto.sh/llms-full.txt)

## Binaries

### Required

- **[Node.js](https://nodejs.org/en/download/):** `>= 22.5`
- **[pnpm](https://pnpm.io/installation):** `>= 10`

### Optional: For Tests

- **[Docker](https://docs.docker.com/get-docker):** `>= 27.0.0`
- **[Playwright](https://playwright.dev/docs/intro#installing-playwright):** `>= 1.51`
- **[Foundry](https://getfoundry.sh/introduction/installation):** `>= 1`

## Commands

### Install

```bash
pnpm install                   # Install dependencies
```

### Development

```bash
pnpm build                     # Build the core library
pnpm dev                       # Link library & run dev servers (playground, dialog, id)
pnpm dev:anvil                 # Run `pnpm dev` with local Relay and Anvil services
pnpm dev:wagmi                 # Run `pnpm dev` with wagmi playground
```

### Testing

```bash
foundryup --install nightly    # Foundry setup

pnpm build:contracts           # Build Contracts with Foundry (optional)
pnpm build:anvil-state         # Build Anvil State (optional)
pnpm test                      # Run default test suite with Vitest
pnpm test:browser              # Run browser-based tests (Chromium, Firefox)
```

### Code Quality

```bash
pnpm check                    # Run Biome linter and formatter
pnpm check:types              # Type checking across all packages
pnpm check:build              # Validate build outputs with publint and attw
pnpm check:size               # Check bundle size limits
pnpm check:repo               # Check repository structure with sherif
pnpm check:knip               # Check for unused dependencies with knip
```

### Contract Development

```bash
foundryup --install nightly    # Foundry setup

# Account contracts
forge build --config-path ./contracts/account/foundry.toml
forge test --config-path ./contracts/account/foundry.toml

# Demo contracts  
forge build --config-path ./contracts/demo/foundry.toml
forge test --config-path ./contracts/demo/foundry.toml
```

## Directory Structure

- `apps/`: Applications
- `contracts/`: Contracts
  - The Porto Account contracts are a submodule under the `account/` directory
- `src/`: `porto` library source
  - Files are represented as modules
  - Files (modules) that are not exposed to the public API are stored in an `internal/` directory
- `scripts/`: Development scripts
- `test/`: Test configuration and utilities

## Project Overview

### Library (`src/`)

#### External Dependencies

- **`viem`** - TypeScript Interface for Ethereum
- **`wagmi`** - React Hooks for Ethereum
- **`ox`** - Standard Library for Ethereum
- **`effect/schema`** - Runtime schema validation

#### Core Modules (`src/core/`)

- **`Porto`** - Main stateful Porto module that links together all other modules
- **`Chains`** - Supported Porto chains
- **`Dialog`** - Dialog instances
- **`Messenger`** - Cross-document communication
- **`Mode`** - Account orchestration mode (e.g. `dialog`, `relay`, `contract`)
- **`Storage`** - Storage interfaces
- **`RpcSchema`** - RPC method schemas and types
- Internal modules are stored in an `internal/` directory

#### Viem Modules (`src/viem/`)

- **`Account`** - Viem Account abstraction for Porto Accounts
- **`Key`** - Account key management and cryptographic operations (e.g. `secp256k1`, `p256`, `webauthn-p256`)
- **`ContractActions`** - Actions for the Ithaca Account contract.
- **`RelayActions`** - Actions for the Relay
- **`RelayClient`** - Client helpers for the Relay
- **`WalletActions`** - Actions for the Porto Dialog (EIP-1193 Provider)
- **`WalletClient`** - Client helpers for the Porto Dialog (EIP-1193 Provider)

#### Wagmi Modules (`src/wagmi/`)

- **`Actions`** - Wagmi Actions
- **`Connector`** - Wagmi Connector
- **`Hooks`** - React Hooks
- **`Query`** - Wagmi Query utilities

### Applications (`apps/`)

- **`docs/`** - Documentation site for the SDK (library), Relay and Contracts
- **`dialog/`** - Dialog application (`id.porto.sh/dialog`)
- **`playground/`** - Development playground
- **`id/`** - Account management application (`id.porto.sh`)
- **`wagmi/`** - Wagmi development playground
- **`verify/`** - Application verification worker


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ithacaxyz/porto](https://github.com/ithacaxyz/porto) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-31 -->
