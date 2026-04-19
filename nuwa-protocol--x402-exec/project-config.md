---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

x402x (x402-exec) is a programmable settlement framework for the x402 protocol. It extends x402 payments with Hook-based business logic execution, facilitator incentives, and atomic transactions. The core innovation is a "commitment-as-nonce" scheme where a cryptographic hash of all settlement parameters becomes the EIP-3009 nonce, binding business logic to user signatures and preventing parameter tampering.

## Repository Structure

This is a **pnpm workspace monorepo** with the following components:

```
x402-exec/
├── contracts/              # Foundry Solidity project
│   ├── src/
│   │   ├── SettlementRouter.sol    # Core settlement contract
│   │   └── interfaces/             # ISettlementRouter, ISettlementHook, IERC3009
│   ├── examples/                   # Hook example contracts (nft-mint, reward-points)
│   ├── script/                     # Deployment scripts
│   └── test/                       # Foundry tests (*.t.sol)
├── typescript/packages/
│   ├── extensions/     # x402 v2 protocol extensions for settlement (commitment calc, networks, middleware)
│   ├── client/         # Client SDK (React/wagmi hooks for browser wallets)
│   └── facilitator-sdk/   # Utilities for facilitator implementations
├── facilitator/        # Production facilitator service (Node/Express)
├── examples/showcase/
│   ├── server/         # Resource server (Express backend demo)
│   └── client/         # Payment client (React + Vite frontend demo)
├── web/
│   ├── frontend/       # Explorer/scanner web UI
│   └── backend/        # Web backend service
├── deps/               # Git submodule for x402 upstream (avoid direct edits)
└── docs/               # Architecture and integration docs
```

## Development Commands

### Setup (First Time)

```bash
# Initialize git submodules (x402 upstream dependency)
git submodule update --init --recursive

# Install all workspace dependencies
pnpm install

# Copy environment template and configure
cp env.template .env
# Edit .env with RPC URLs, API keys, etc.
```

### Build Commands

```bash
pnpm build              # Build SDK packages + examples + facilitator
pnpm build:sdk          # Build only TypeScript packages (typescript/packages/*)
pnpm build:facilitator  # Build facilitator service
pnpm build:showcase     # Build showcase demo
```

### Testing

```bash
# Solidity contracts (Foundry)
cd contracts
forge build
forge test                         # Run all tests
forge test --gas-report            # With gas reporting
forge coverage                     # Coverage report

# TypeScript packages (Vitest)
pnpm --filter './typescript/packages/**' test       # All SDK tests
pnpm --filter @x402x/extensions test               # Specific package
pnpm --filter @x402x/extensions test:coverage

# Facilitator
pnpm -C facilitator test
pnpm -C facilitator test:unit
pnpm -C facilitator test:e2e
```

### Development Servers

```bash
pnpm dev                    # Showcase app (client + server)
pnpm dev:server             # Showcase server only (port 3000)
pnpm dev:client             # Showcase client only (port 5173)
pnpm dev:facilitator        # Facilitator service
pnpm -C web/frontend dev    # Web UI
pnpm -C web/backend dev     # Web backend
```

### Code Quality

```bash
pnpm format                 # Format TypeScript (Prettier, 2-space, ~100 cols)
pnpm format:check
pnpm -C web/frontend check  # Web frontend uses Biome
```

### Contract Deployment

```bash
cd contracts
./deploy-contract.sh [NETWORK] [OPTIONS]

# Examples (using CAIP-2 format):
./deploy-contract.sh eip155:84532 --all --verify    # Deploy all contracts (Base Sepolia)
./deploy-contract.sh eip155:8453 --settlement --verify     # SettlementRouter only (Base Mainnet)
./deploy-contract.sh eip155:196 --hooks --verify        # Built-in hooks (X-Layer Mainnet)
```

## Key Architecture Concepts

### Commitment-as-Nonce Scheme

The cryptographic foundation of x402x. The client computes a commitment hash from **all** settlement parameters and uses it as the EIP-3009 nonce:

```typescript
nonce = keccak256(
  "X402/settle/v1",
  chainId,
  hub,              // SettlementRouter address
  token, from, value,
  validAfter, validBefore,
  salt,
  payTo,            // Final recipient (for transparency)
  facilitatorFee,
  hook,             // Hook contract address
  keccak256(hookData)
);
```

This binds business parameters to the signature, prevents parameter tampering, and enables stateless facilitator processing.

### Settlement Flow

1. **Client** receives `PaymentRequirements` with `extra.settlementRouter` from resource server
2. **Client** computes commitment, signs EIP-3009 `TransferWithAuthorization` (with nonce=commitment, to=router)
3. **Facilitator** receives `X-PAYMENT` header, detects settlement mode, calls `SettlementRouter.settleAndExecute()`
4. **SettlementRouter** verifies commitment, executes EIP-3009 transfer, deducts fee, calls Hook
5. **Hook** executes business logic (split, mint NFT, distribute points, etc.)
6. **SettlementRouter** verifies zero balance (no fund holding), emits events

### Hook Interface

```solidity
interface ISettlementHook {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/nuwa-protocol) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
