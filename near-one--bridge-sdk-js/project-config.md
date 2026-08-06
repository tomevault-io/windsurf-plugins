---
trigger: always_on
description: bun run build          # TypeScript compilation to dist/
---

# Repository Guidelines

## Development Commands

```bash
# Build and development
bun run build          # TypeScript compilation to dist/
bun run typecheck      # Type checking without build
bun run lint           # Biome linting and formatting

# Testing
bun run test           # Run all tests with Vitest
bun run test <pattern> # Run specific test files matching pattern
bun run test --watch   # Watch mode for development
bun run test e2e/      # Run e2e tests (requires funded testnet accounts)

# Package management
bun install            # Install dependencies
bun run check-exports  # Validate package exports with @arethetypeswrong/cli
```

## Architecture Overview

This is a monorepo containing the `@omni-bridge/*` SDK packages under `packages/`. The SDK uses a **transaction builder pattern** - it handles all bridge protocol logic (validation, encoding, fee calculation) but returns unsigned transactions for consumers to sign and broadcast with their own tooling.

### Package Structure

```
packages/
├── core/        # @omni-bridge/core - Types, validation, config, API client
├── evm/         # @omni-bridge/evm - EVM transaction builder (viem-based)
├── near/        # @omni-bridge/near - NEAR transaction builder + shims
├── solana/      # @omni-bridge/solana - Solana instruction builder (Anchor-based)
├── starknet/    # @omni-bridge/starknet - Starknet transaction builder
├── aptos/       # @omni-bridge/aptos - Aptos entry-function payload builder
├── btc/         # @omni-bridge/btc - Bitcoin/Zcash UTXO operations
└── sdk/         # @omni-bridge/sdk - Umbrella re-export of all packages
```

### Core Concepts

**Factory Pattern**: Each chain has a builder factory:

- `createBridge({ network })` → validation and API access
- `createEvmBuilder({ network, chain })` → EVM transaction building
- `createNearBuilder({ network })` → NEAR transaction building
- `createSolanaBuilder({ network, connection? })` → Solana instruction building
- `createStarknetBuilder({ network, bridgeAddress? })` → Starknet Call[] building
- `createAptosBuilder({ network, bridgeAddress? })` → Aptos entry-function payloads
- `createBtcBuilder({ network, chain })` → Bitcoin/Zcash UTXO operations

**Unsigned Transaction Types**: SDK returns library-agnostic plain objects:

- `EvmUnsignedTransaction` → Compatible with viem and ethers v6 directly
- `NearUnsignedTransaction` → Use shims: `toNearKitTransaction()` or `sendWithNearApiJs()`
- `TransactionInstruction[]` → Native @solana/web3.js instructions
- `Call[]` → Native starknet.js calls
- `AptosFunctionPayload` → Compatible with @aptos-labs/ts-sdk `InputEntryFunctionData`
- `BtcWithdrawalPlan` → UTXO inputs/outputs for signing

**OmniAddress System**: Cross-chain addresses use chain prefixes:
`eth:0x...`, `near:account.near`, `sol:...`, `base:0x...`, `arb:0x...`, `strk:0x...`, `aptos:0x...`, `btc:...`, `zcash:...`

### Transfer Flow

1. `bridge.validateTransfer(params)` → Validates and returns `ValidatedTransfer`
2. `builder.buildTransfer(validated)` → Returns unsigned transaction
3. Consumer signs and broadcasts using their preferred library

### Key Files

- `packages/core/src/bridge.ts` - `createBridge()` factory and validation
- `packages/core/src/types.ts` - Core types (`OmniAddress`, `ValidatedTransfer`, unsigned tx types)
- `packages/core/src/api.ts` - REST API client with Zod validation
- `packages/core/src/config.ts` - Network addresses and chain IDs
- `packages/evm/src/builder.ts` - EVM transaction builder
- `packages/near/src/builder.ts` - NEAR transaction builder
- `packages/near/src/shims.ts` - near-kit and near-api-js conversion helpers
- `packages/solana/src/builder.ts` - Solana instruction builder
- `packages/starknet/src/builder.ts` - Starknet transaction builder
- `packages/aptos/src/builder.ts` - Aptos payload builder
- `packages/btc/src/builder.ts` - Bitcoin/Zcash UTXO builder

## Testing Patterns

Tests use Vitest with MSW for API mocking:

- **Unit tests**: `packages/*/tests/*.test.ts` - Pure function testing
- **E2E tests**: `e2e/*.test.ts` - Real testnet transactions

Run specific package tests:

```bash
bun run test packages/core/    # Core package tests
bun run test packages/evm/     # EVM builder tests
```

## Code Style

- **Biome** formatting: 2-space indents, 100-char line width, double quotes, no semicolons
- **Import extensions**: Must use `.js` extensions in imports (Biome rule)
- **No Node.js APIs**: Use `Uint8Array`/`TextEncoder` instead of `Buffer`, `@noble/hashes` instead of `crypto`
- **Strict TypeScript**: Full type safety with strict compiler options
- **ESM modules**: Uses Node.js ESM with `.js` extension requirement

## Workflow Rules

### Documentation Updates

When making changes to the SDK, always update documentation accordingly:

- **Reference docs** (`docs/reference/`): Update when adding/changing public API methods, parameters, or return types
- **Code snippets**: Ensure all code examples in docs still compile and reflect current API
- **Package READMEs** (`packages/*/README.md`): Keep API sections in sync with actual exports
- **Guides** (`docs/guides/`): Update when workflows or best practices change
- **Examples** (`docs/examples/`, `examples/`): Verify examples work with any API changes


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Near-One/bridge-sdk-js](https://github.com/Near-One/bridge-sdk-js) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-06 -->
