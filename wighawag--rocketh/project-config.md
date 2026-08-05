---
trigger: always_on
description: A framework-agnostic smart contract deployment system for Ethereum-compatible networks.
---

# Rocketh - AI Agent Instructions

A framework-agnostic smart contract deployment system for Ethereum-compatible networks.

---

## Tech Stack

- **Language:** TypeScript (strict mode, ESNext target)
- **Module System:** ESM (ES Modules) with NodeNext resolution
- **Build System:** Nx monorepo with pnpm workspaces
- **Testing:** Vitest (unit tests + integration tests as documentation)
- **Formatting:** Prettier
- **Node Version:** 22.14.0 (via Volta)
- **Package Manager:** pnpm 10.28.1

---

## Project Structure

```
rocketh/
├── packages/
│   ├── rocketh/              # Core deployment environment and execution
│   ├── rocketh-core/         # Shared types and utilities
│   ├── rocketh-deploy/       # Standard contract deployment
│   ├── rocketh-proxy/        # Proxy deployment patterns (UUPS, Transparent, ERC173)
│   ├── rocketh-diamond/      # EIP-2535 Diamond proxy support
│   ├── rocketh-node/         # Node.js deployment executor (filesystem access)
│   ├── rocketh-verifier/     # Contract verification (Etherscan, Sourcify)
│   ├── rocketh-export/       # Export deployments for frontend consumption
│   ├── rocketh-doc/          # Documentation generation
│   ├── rocketh-signer/       # Signer utilities
│   ├── rocketh-router/       # Route-based contract deployment
│   ├── rocketh-viem/         # Viem integration
│   ├── rocketh-web/          # Browser runtime support
│   ├── rocketh-read-execute/ # Contract read/write utilities
│   └── rocketh-test-utils/   # Test utilities and mock helpers
├── documentation.md          # Main documentation
├── TESTING.md               # Testing guide
└── README.md                # Getting started guide
```

### Key Files Reference

- **Entry points:** `packages/*/src/index.ts`
- **Types:** `packages/rocketh-core/src/types.ts`
- **Deploy function:** `packages/rocketh-deploy/src/index.ts`
- **Proxy deployments:** `packages/rocketh-proxy/src/index.ts`
- **Diamond deployments:** `packages/rocketh-diamond/src/index.ts`
- **Test utilities:** `packages/rocketh-test-utils/src/index.ts`

---

## Commands

```bash
# Install dependencies
pnpm install

# Build all packages
pnpm build
# or
nx run-many -t build

# Type check all packages
pnpm typecheck
# or single file
pnpm -r --parallel exec tsc --noEmit

# Run all tests
pnpm test

# Run tests in watch mode
pnpm test:watch

# Run tests with UI
pnpm test:ui

# Run tests with coverage
pnpm test:coverage

# Format code
pnpm format

# Check formatting
pnpm format:check

# Lint a specific file
pnpm prettier --write path/to/file.ts

# Run docs development server
pnpm docs:dev

# Build docs
pnpm docs:build
```

---

## Do

- Use TypeScript with strict mode for all new files
- Use ESM imports with `.js` extensions (e.g., `import {deploy} from './deploy.js'`)
- Use `type` imports when importing only types: `import type {Environment} from '@rocketh/core/types'`
- Use `viem` for Ethereum utilities (encoding, addresses, etc.)
- Use `named-logs` for logging: `const logger = logs('@rocketh/package-name')`
- Use `abitype` for ABI typing with `Abi` type
- Use `eip-1193` types for EIP-1193 provider interactions
- Follow existing patterns in similar files when adding new functionality
- Write integration tests that serve as documentation (see `packages/*/test/*.integration.test.ts`)
- Use `createMockEnvironment` and `createMockArtifact` from `@rocketh/test-utils` for tests
- Keep functions focused and modular - one concern per function
- Export types separately from implementation: `export type * from './types.js'`
- Use `as const satisfies` pattern for configuration objects
- Keep deployment logic separate from environment setup
- Use descriptive error messages that help users debug issues

---

## Don't

- Do not use CommonJS (`require`, `module.exports`)
- Do not import without `.js` extension in relative imports
- Do not modify `@rocketh/core` types without considering impacts on all packages
- Do not add new heavy dependencies without approval
- Do not hardcode addresses - use named accounts or environment configuration
- Do not use `any` type - use proper types or `unknown` with type guards
- Do not mix environment setup logic with deployment logic
- Do not use class-based patterns - prefer functional patterns with currying
- Do not skip tests for new functionality
- Do not modify hardhat-deploy-v1-artifacts unless necessary for compatibility
- Do not use sync filesystem operations in packages that should work in browser

---

## Code Style Examples

### Good: Deploy function pattern (curried)

```typescript
// packages/rocketh-deploy/src/index.ts
export function deploy(env: Environment): <TAbi extends Abi>(
  name: string,
  args: DeploymentConstruction<TAbi>,
  options?: DeployOptions,
) => Promise<DeployResult<TAbi>> {
  return async <TAbi extends Abi>(name: string, args: DeploymentConstruction<TAbi>, options?: DeployOptions) => {
    // Implementation
  };
}
```

### Good: Type exports pattern

```typescript
// packages/rocketh-core/src/index.ts
export type * from './types.js';
export {resolveAccount, resolveAccountOrUndefined} from './account.js';
export {mergeABIs, mergeArtifacts} from './artifacts.js';
```

### Good: Test structure pattern

```typescript

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wighawag/rocketh](https://github.com/wighawag/rocketh) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
