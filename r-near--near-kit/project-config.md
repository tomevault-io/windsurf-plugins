---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

near-kit is a TypeScript monorepo for interacting with NEAR Protocol, designed to be simple and intuitive - like a modern fetch library.

**Packages:**

- `near-kit` - Core TypeScript library for NEAR Protocol interactions
- `@near-kit/react` - React bindings with hooks and providers (planned)

**Core Principles:**

- **Simple things should be simple** - One-line commands for common operations
- **Type safety everywhere** - Full TypeScript support with IDE autocomplete
- **Human-readable** - Use "10 NEAR" not "10000000000000000000000000" (yoctoNEAR)
- **Progressive complexity** - Basic API for simple needs, advanced features when required

## Monorepo Structure

```
near-kit/
├── packages/
│   ├── near-kit/          # Core library
│   │   ├── src/
│   │   ├── tests/
│   │   ├── package.json
│   │   └── tsconfig.json
│   └── react/             # React bindings (planned)
├── package.json           # Root workspace config
├── tsconfig.json          # Shared TypeScript config
└── biome.json             # Shared linting config
```

## Development Commands

### Setup

```bash
bun install
```

### Testing

Tests use [Vitest](https://vitest.dev/) (not Bun's test runner) which runs tests in parallel per-file for faster integration test execution.

```bash
# Run all tests across all packages
bun run test

# Run tests for a specific package
bun run --filter near-kit test

# Run specific test types (from packages/near-kit)
cd packages/near-kit
bun run test:unit               # Unit tests only
bun run test:integration        # Integration tests only
```

### Building & Type Checking

```bash
bun run build              # Build all packages
bun run typecheck          # Type check all packages
bun run lint               # Lint and format all packages with Biome
```

### Package-specific commands

```bash
bun run --filter near-kit build      # Build specific package
bun run --filter near-kit typecheck  # Typecheck specific package
```

## Changesets

This project uses [changesets](https://github.com/changesets/changesets) for version management across packages.

**IMPORTANT:** When making changes that should be included in the next release, you MUST create a changeset file manually. The interactive CLI (`bun changeset`) does not work in AI environments.

### Creating a Changeset

Create a new markdown file in `.changeset/` directory:

```bash
# File: .changeset/descriptive-name.md
---
"near-kit": patch  # or "minor" or "major"
---

Brief description of the change
```

For changes affecting multiple packages:

```markdown
---
"near-kit": minor
"@near-kit/react": minor
---

Add new feature across core and react packages
```

**Change types:**

- `patch` - Bug fixes, minor improvements (0.0.X)
- `minor` - New features, backwards-compatible (0.X.0)
- `major` - Breaking changes (X.0.0)

## Architecture

### Core API Structure

The library is built around a main `Near` class with three interaction patterns:

1. **Simple operations** - Direct methods: `near.view()`, `near.call()`, `near.send()`
2. **Transaction builder** - Fluent API: `near.transaction()` for multi-action transactions
3. **Type-safe contracts** - Typed proxies: `near.contract<T>()` with full IDE autocomplete

### Key Modules (packages/near-kit/src/)

**`core/near.ts` - Main Client**

- Central entry point for all operations
- Configuration resolution (networks, keystores, signers, wallets)
- Async keystore initialization support
- Auto-detects sandbox root account keys

**`core/transaction.ts` - TransactionBuilder**

- Fluent API for chaining actions: `transfer()`, `functionCall()`, `createAccount()`, etc.
- Signing pipeline: `.build()` → `.sign()` → `.send()`
- Automatic nonce management with retry logic (3x for InvalidNonceError)
- Supports delegate actions (NEP-366) via `.delegate()`

**`core/rpc/` - RPC Client**

- Low-level NEAR JSON-RPC interface
- Automatic retries with exponential backoff (default: 4 retries, 1s initial delay)
- Error classification and mapping to typed exceptions
- Zod schema validation for all RPC responses

**`core/nonce-manager.ts` - Concurrent Transaction Support**

- Prevents nonce collisions for concurrent transactions
- Local nonce caching with deduplication
- Invalidation support for retry scenarios
- Shared static instance used by TransactionBuilder

**`keys/` - Key Management**

- `InMemoryKeyStore` - Ephemeral runtime storage
- `FileKeyStore` - NEAR-CLI compatible file storage (`~/.near-credentials/`)
- `NativeKeyStore` - OS keyring integration (macOS Keychain, Windows Credential Manager)
- `RotatingKeyStore` - High-throughput concurrent transactions (round-robin key rotation)

**`contracts/contract.ts` - Type-Safe Contract Interface**

- Dynamic proxy creation for typed contract methods
- Splits methods into `view` (free) and `call` (costs gas)
- Full TypeScript inference and IDE autocomplete

**`errors/` - Error Hierarchy**

- All errors extend `NearError` with `code` and optional `data`
- Categories: Blockchain state, transaction failures, contract execution, network issues, validation
- `retryable` flag indicates safe-to-retry operations


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [r-near/near-kit](https://github.com/r-near/near-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
