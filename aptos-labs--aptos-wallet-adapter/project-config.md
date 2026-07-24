---
trigger: always_on
description: This document provides context for AI assistants working with the Aptos Wallet Adapter monorepo.
---

# Aptos Wallet Adapter

This document provides context for AI assistants working with the Aptos Wallet Adapter monorepo.

## Project Overview

The Aptos Wallet Adapter is a comprehensive monorepo for building dapps on Aptos with wallet integration and cross-chain functionality. It provides:

- **Wallet Adapter SDK**: Core functionality for connecting Aptos wallets to dapps
- **React Integration**: React provider and hooks for wallet interaction
- **Cross-Chain Transfers**: USDC transfers between Aptos and other chains via Circle's CCTP
- **Derived Wallets**: Create Aptos wallets from external chain keys (Ethereum, Solana, Sui)
- **UI Components**: Pre-built wallet selectors for Ant Design and Material-UI

## Repository Structure

This is a **Turbo monorepo** with two main workspaces:

```
aptos-wallet-adapter/
├── apps/                          # Demo applications
│   ├── nextjs-example/           # Basic wallet adapter demo
│   ├── nextjs-x-chain/           # Cross-chain transfers demo
├── packages/                      # Published packages
│   ├── wallet-adapter-core/      # Core adapter logic
│   ├── wallet-adapter-react/     # React provider and hooks
│   ├── wallet-adapter-ant-design/ # Ant Design UI components
│   ├── wallet-adapter-mui-design/ # Material-UI components
│   ├── cross-chain-core/         # Cross-chain USDC transfers SDK
│   ├── derived-wallet-base/      # Base for derived wallets
│   ├── derived-wallet-ethereum/  # Ethereum derived wallet
│   ├── derived-wallet-solana/    # Solana derived wallet
│   └── derived-wallet-sui/       # Sui derived wallet
├── turbo.json                     # Turbo build configuration
├── package.json                   # Root package with workspaces
└── pnpm-workspace.yaml           # pnpm workspace configuration
```

## Key Packages

### Core Packages

**wallet-adapter-core**
- Core adapter state management and wallet interaction logic
- Wallet registration and connection handling
- Network configuration (mainnet/testnet/devnet)
- Does not include UI components

**wallet-adapter-react**
- React Context provider (`AptosWalletAdapterProvider`)
- React hooks: `useWallet()`, `useWalletConnect()`, etc.
- Depends on wallet-adapter-core
- Entry point for most React dapps

### Cross-Chain Packages

**cross-chain-core**
- SDK for cross-chain USDC transfers via Wormhole and Circle's CCTP
- Supports transfers between Aptos and: Solana, Ethereum, Sui, Base, Arbitrum, Avalanche, Polygon
- Two-phase transfer process: initiate (burn) and claim (mint)
- Uses derived wallets for seamless onboarding

**derived-wallet-{ethereum,solana,sui}**
- Create Aptos accounts derived from external chain keys
- Enables users to control Aptos assets with their existing wallets
- Each package implements chain-specific signing and key derivation

## Development Workflow

### Requirements
- Node.js 20.18.0+
- pnpm 9.15.5

### Common Commands

```bash
# Install dependencies
pnpm install

# Build all packages
pnpm turbo run build

# Run dev server (starts nextjs-example on https://localhost:3000)
pnpm turbo run dev

# Run tests
pnpm test

# Run tests for specific package
cd packages/wallet-adapter-react && pnpm test

# Clean build artifacts
pnpm turbo run clean

# Create changeset for version bump
pnpm changeset
```

### Build Order

Turbo handles build dependencies automatically via `dependsOn` in `turbo.json`. Packages are built in order:
1. Base packages (tsconfig, eslint-config-adapter)
2. Core packages (wallet-adapter-core, derived-wallet-base)
3. Integration packages (wallet-adapter-react, derived wallets)
4. UI packages (ant-design, mui-design)
5. Apps (nextjs-example, nextjs-x-chain)

## Testing

All packages use **Vitest** for testing with comprehensive coverage:

- **wallet-adapter-core**: Core adapter functionality, wallet state management
- **wallet-adapter-react**: React hooks, provider, components
- **cross-chain-core**: Cross-chain transfer flows, signers, providers
- **derived-wallet-{ethereum,solana,sui}**: Key derivation, signing, message formatting

Tests are located in `tests/` directories within each package.

### Test Setup
- Uses `happy-dom` for DOM simulation in React tests
- Mocks wallet providers and external dependencies
- Tests both happy paths and error scenarios

## Code Conventions

### Wallet Adapter Standard (AIP-62)
The adapter implements the Aptos Wallet Standard (AIP-62):
- Wallets expose `signAndSubmitTransaction`, `signMessage`, `account`, `network`
- Dapps interact via standardized interface
- Automatic wallet detection and connection

### Key Concepts

**Wallet Registration**
- Wallets register themselves with the adapter
- Adapter maintains wallet registry and connection state
- Users select wallet from UI or programmatically

**Network Configuration**
- Support for mainnet, testnet, devnet
- Each wallet reports its active network
- Dapps can enforce network requirements

**Transaction Signing**
- `signAndSubmitTransaction`: Sign and submit to chain
- `signTransaction`: Sign without submitting
- `signMessage`: Sign arbitrary messages

**Cross-Chain Signers**
- Implement Wormhole's `SignAndSendSigner` interface
- Handle chain-specific transaction formatting
- `AptosSigner`: User-interactive via wallet adapter

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aptos-labs/aptos-wallet-adapter](https://github.com/aptos-labs/aptos-wallet-adapter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
