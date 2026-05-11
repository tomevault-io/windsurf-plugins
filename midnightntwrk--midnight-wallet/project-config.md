---
trigger: always_on
description: provides a solution:
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

The Midnight Wallet SDK is a TypeScript implementation of the
[Midnight Wallet Specification](https://github.com/midnightntwrk/midnight-architecture/blob/main/components/WalletEngine/Specification.md).
It provides key generation, address formatting, transaction building, state syncing with the indexer, and testing
utilities for the Midnight privacy-focused blockchain.

## Claude Code Settings

- **`.claude/settings.json`** is tracked by git — shared team config (hooks only). **NEVER** add `permissions` here.
- **`.claude/settings.local.json`** is gitignored — personal permissions go here.

## Key Specifications (ALWAYS CONSULT)

When working on wallet functionality, always consult these specifications:

### Wallet Specification

**Repository:** [midnight-architecture](https://github.com/midnightntwrk/midnight-architecture) **Path:**
`components/WalletEngine/Specification.md`

Key sections:

- Transaction lifecycle: pending → confirmed → finalized (or discarded)
- Coin lifecycle: booked → pending → confirmed → final
- Balance types: available, pending, total
- State operations: apply_transaction, finalize_transaction, discard_transaction, spend
- Synchronization process and indexing services

### DApp Connector API Specification

**Repository:** [midnight-dapp-connector-api](https://github.com/midnightntwrk/midnight-dapp-connector-api) **NPM:**
[@midnight-ntwrk/dapp-connector-api](https://www.npmjs.com/package/@midnight-ntwrk/dapp-connector-api) **Path:**
`SPECIFICATION.md`

Key sections:

- API design philosophy and responsibilities
- Method signatures and expected behaviors
- Error handling requirements
- Transaction structure requirements

### DApp Connector API Types

**Path:** `src/api.ts` in the dapp-connector-api package

TypeScript type definitions for the connector API.

### Ledger Specification

**Repository:** [midnight-ledger](https://github.com/midnightntwrk/midnight-ledger) **Path:** `spec/`

Key documents:

- `intents-transactions.md` - Transaction structure, intents, sections
- `zswap.md` - Shielded token protocol
- `dust.md` - Dust token mechanics
- `night.md` - Night/unshielded token mechanics
- `contracts.md` - Smart contract execution
- `cost-model.md` - Transaction fee calculation

### API Usage Examples

**Package:** `packages/docs-snippets`

Contains working code examples for common wallet operations:

- `combined-transfer.ts` - Transfer both shielded and unshielded tokens
- `shielded-transfer.ts` - Shielded token transfer
- `unshielded-transfer.ts` - Unshielded token transfer
- `swap.ts` - Token swap (intent creation)
- `balancing.ts` - Transaction balancing
- `initialization.ts` - Wallet initialization

**IMPORTANT:** Always refer to docs-snippets for API usage patterns when implementing new features.

## Build Commands and tools in use

All commands must be run from the repository root. Do not cd into a package directory to run commands — shared
devDependencies (vitest, typescript, eslint, etc.) are hoisted to the root node_modules and won't resolve from
individual package directories. Use --filter to target specific packages.

```bash
# Setup (use nvm or nix develop with direnv)
nvm use && corepack enable

# Install dependencies
yarn

# Build all packages
yarn dist

# Build specific package
yarn dist --filter=@midnight-ntwrk/wallet-sdk-facade

# Build and watch for changes
yarn watch

# Run all unit tests
yarn test

# Run tests for specific package
yarn test --filter=@midnight-ntwrk/wallet-sdk-unshielded-wallet

# Run specific test file
yarn test --filter=@midnight-ntwrk/wallet-sdk-unshielded-wallet -- test/UnshieldedWallet.test.ts

# Full CI verification (typecheck, lint, tests)
yarn verify

# Check/fix formatting
yarn format:check
yarn format

# Clean all build artifacts
yarn clean

# Add a changeset for versioning
yarn changeset add

# Check for missing changesets
yarn changeset:check

# --- Effect Language Service (see section below) ---
# Run Effect diagnostics on a specific file
yarn effect-language-service diagnostics --file "$(pwd)/path/to/file.ts" --format pretty

# Run Effect diagnostics on a whole package (must use tsconfig.build.json or tsconfig.test.json, NOT tsconfig.json — the latter uses references with no source files)
yarn effect-language-service diagnostics --project "$(pwd)/packages/dust-wallet/tsconfig.build.json" --format pretty

# Show quickfixes (report-only, does not auto-apply) on a specific file
yarn effect-language-service quickfixes --file "$(pwd)/path/to/file.ts"

```

### Effect Language Service

The project uses `@effect/language-service` for Effect-specific diagnostics, quickfixes, and code quality checks. It is
configured in `tsconfig.base.json` as a TypeScript plugin.

**CLI commands** (all prefixed with `yarn effect-language-service`):

- `diagnostics` — Report Effect-specific issues (floating effects, wrong yield usage, deterministic keys, etc.)
- `quickfixes` — Show diagnostics with proposed code diffs (report-only, does NOT auto-apply fixes)
- `codegen` — Apply `@effect-codegens` directive transformations (this one DOES write changes)
- `overview` — Summarize Effect exports (services, layers, errors) in a file or project

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [midnightntwrk/midnight-wallet](https://github.com/midnightntwrk/midnight-wallet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
