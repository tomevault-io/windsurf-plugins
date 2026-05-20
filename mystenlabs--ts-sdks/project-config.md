---
trigger: always_on
description: This file provides guidance to AI agents working with code in this repository.
---

# AGENTS.md

This file provides guidance to AI agents working with code in this repository.

## Overview

This is a monorepo containing TypeScript SDKs for the Sui blockchain ecosystem. It uses pnpm workspaces, turbo for build orchestration, and includes packages for core Sui functionality, dApp development, wallet integration, and various blockchain services.

## Common Commands

### Setup and Build

```bash
# Initial setup
pnpm install
pnpm turbo build

# Build all packages
pnpm build

# Build a specific package with dependencies
pnpm turbo build --filter=@mysten/sui
```

### Testing

```bash
# Run unit tests
pnpm test

# Run unit tests for a specific package
pnpm --filter @mysten/sui test

# Run a single test file
pnpm --filter @mysten/sui vitest run path/to/test.spec.ts

# Run e2e tests (requires Docker for local network)
# All e2e tests for a package:
pnpm --filter @mysten/sui vitest run --config test/e2e/vitest.config.mts

# A specific e2e test file:
pnpm --filter @mysten/sui vitest run --config test/e2e/vitest.config.mts test/e2e/clients/core/objects.test.ts
```

### Linting and Formatting

```bash
# Check lint and formatting
pnpm lint

# Auto-fix lint and formatting issues
pnpm lint:fix

# Run oxlint and prettier separately
pnpm oxlint:check
pnpm prettier:check
```

### Package Management

```bash
# Add a changeset for version updates
pnpm changeset

# Version packages
pnpm changeset-version
```

## Architecture

### Repository Structure

- **packages/** - All SDK packages organized by functionality
  - **typescript/** - Core Sui SDK with submodules for bcs, client, cryptography, transactions, etc.
  - **dapp-kit/** - React hooks and components for dApp development
  - **wallet-standard/** - Wallet adapter implementation
  - **signers/** - Various signing solutions (AWS KMS, GCP KMS, Ledger, etc.)
  - **suins/** - Sui Name Service integration
  - **deepbook/** - DEX integration packages
  - **zksend/** - zkSend functionality

### Build System

- Uses Turbo for monorepo task orchestration with dependency-aware builds
- Each package can have its own test configuration (typically using Vitest)
- Common build outputs: `dist/` for compiled code, with both ESM and CJS formats

### Key Patterns

1. **Modular exports**: Packages use subpath exports (e.g., `@mysten/sui/client`, `@mysten/sui/bcs`)
2. **Shared utilities**: Common functionality in `packages/utils`
3. **Code generation**: Some packages use GraphQL codegen and version generation scripts
4. **Testing**: Unit tests alongside source files, e2e tests in separate directories
5. **Type safety**: Extensive TypeScript usage with strict type checking

### Sui Client Architecture (`packages/sui`)

The `@mysten/sui` package has a multi-transport client architecture. Understanding its layered design is critical before making changes.

#### Layered Client Design

The client system has three layers:

1. **Public client** (`SuiGrpcClient`, `SuiGraphQLClient`, `SuiJsonRpcClient`) — what users instantiate. Provides transport-specific "Native API" access (e.g., raw gRPC service clients, raw GraphQL queries) plus the unified `client.core` property. Supports extension via `$extend`.

2. **Core implementation** (`GrpcCoreClient`, `GraphQLCoreClient`, `JSONRpcCoreClient`) — each extends the abstract `CoreClient` and maps protocol-specific wire data (protobuf, GraphQL fragments, JSON) into unified `SuiClientTypes`. This is where most business logic lives.

3. **Abstract contract** (`CoreClient` in `src/client/core.ts`) — defines the "Core API" that all transports implement. Also provides transport-agnostic composed methods (e.g., `getObject` delegates to `getObjects`, `getDynamicField` uses `getObjects` + BCS parsing).

Key files:

| Layer             | gRPC                  | GraphQL                 | JSON-RPC                |
| ----------------- | --------------------- | ----------------------- | ----------------------- |
| Public client     | `src/grpc/client.ts`  | `src/graphql/client.ts` | `src/jsonRpc/client.ts` |
| Core impl         | `src/grpc/core.ts`    | `src/graphql/core.ts`   | `src/jsonRpc/core.ts`   |
| Abstract contract | `src/client/core.ts`  | ←                       | ←                       |
| Shared types      | `src/client/types.ts` | ←                       | ←                       |

#### Cross-Client Consistency

All three transports must produce identical results for the same Core API call. This is the most important architectural invariant. When making changes:

- **Always read all three implementations** of the affected method before changing any of them.
- A bug in one transport very often exists (in a different form) in the others.
- Each transport has different protocol-level concerns (gRPC read masks, GraphQL query fields, JSON-RPC response shapes) but they must all produce the same unified output.

#### Unified Type System (`src/client/types.ts`)

All Core API methods return types from the `SuiClientTypes` namespace. Key design patterns:

- **Discriminated unions with `$kind`**: All polymorphic types use a `$kind` string literal to discriminate variants. This is used for `ObjectOwner`, `TransactionResult`, `ExecutionError`, `DatatypeResponse`, and others.
  ```typescript

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MystenLabs/ts-sdks](https://github.com/MystenLabs/ts-sdks) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
