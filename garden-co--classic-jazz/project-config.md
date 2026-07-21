---
trigger: always_on
description: Guidelines for AI agents working on the Jazz codebase.
---

# AGENTS.md

Guidelines for AI agents working on the Jazz codebase.

## Project Overview

Jazz is a distributed database framework for building local-first applications. It provides real-time collaboration, offline support, and end-to-end encryption through CRDTs (Conflict-free Replicated Data Types). Data is stored locally and synced peer-to-peer with automatic conflict resolution.

- **Monorepo**: pnpm workspaces with Turbo build orchestration
- **Languages**: TypeScript (primary), Rust (performance-critical CRDT code in `crates/`)
- **Node.js**: 22.18.0+ required
- **Package Manager**: pnpm 10.16.1

### Key Domain Concepts

- **CoValue**: Base collaborative value type — the core data abstraction. Variants: `CoMap` (key-value), `CoList` (ordered list), `CoStream` (append-only stream), `CoPlainText` (collaborative text), `BinaryCoStream` (files/images/audio)
- **Group**: Permission and access control entity that manages read/write access to CoValues. Supports invitations via `InviteSecret`
- **Account**: User identity entity extending Group, with authentication and agent secrets
- **LocalNode**: Entry point for creating and accessing CoValues from a specific account perspective. Manages sync with peers
- **Peer**: A sync connection to another node (server or client)
- **SyncMessage**: Protocol messages for sync: `Load`, `KnownState`, `NewContent`, `Done`
- **CryptoProvider**: Abstract interface for crypto operations (encrypt, sign, derive keys). Implementations: NAPI (Node.js), WASM (browser), RN (mobile)

### Architecture Layers

1. **Core layer** (`cojson`): CRDT operations, sync protocol, storage and crypto abstractions
2. **Native layer** (`crates/`): Rust implementations compiled to NAPI, WASM, and React Native
3. **Tools layer** (`jazz-tools`): Framework bindings (React, Svelte, Vue, RN), schema definitions, high-level APIs
4. **Storage adapters**: Pluggable backends — SQLite, IndexedDB, Durable Objects SQLite
5. **Transport layer**: WebSocket transport (`cojson-transport-ws`) for peer-to-peer sync

## Quick Reference

```bash
pnpm install              # Install dependencies
pnpm build:packages       # Build TypeScript packages
pnpm build:core           # Build native packages (NAPI, WASM, RN)
pnpm build:all-packages   # Build everything (native + TypeScript)
pnpm test                 # Run tests (Vitest, watch mode)
pnpm test --watch=false   # Run tests without watch
pnpm test fileName        # Run tests on files matching fileName
pnpm format-and-lint:fix  # Format and lint (Biome)
pnpm bench                # Run benchmarks
```

### Framework Support

- **React** 19.1.0 — hooks (`useCoState`, `useAccount`, `useJazz`)
- **Svelte** 5.46.4 — stores and components
- **Vue** — community bindings (composables)
- **React Native** 0.81.5 / **Expo** 54.0.23 — mobile support
- **Next.js** / **SvelteKit** 2.49.5 — SSR integration

## Repository Structure

```
packages/       # Main npm packages (jazz-tools, cojson, etc.)
crates/         # Rust code (cojson-core NAPI, WASM, React Native)
examples/       # Example applications
starters/       # Project starter templates
tests/          # Integration and e2e tests
homepage/       # Documentation site (Next.js)
```

## Code Conventions on packages

### Imports

- **Always use `.js` extensions** in imports, even for TypeScript files (enforced by Biome `useImportExtensions` rule at error level)
- Use **relative imports** only — no path aliases are configured
- Use **barrel exports** via `index.ts` files for public APIs

### Naming

- `camelCase` for functions and variables
- `PascalCase` for types, interfaces, classes, and components
- `Raw` prefix for low-level cojson types (e.g., `RawCoMap`, `RawGroup`, `RawAccount`)
- `unstable_` or `experimental_` prefix for unstable APIs

### TypeScript

- **Strict mode** enabled across all packages
- **`noUncheckedIndexedAccess`** enabled
- Avoid `any` types in production code
- **Module resolution**: `bundler`
- **Targets**: ES2020 for `cojson`, ES2021 for `jazz-tools` (React Native/Hermes compatibility)

### Documentation

- Document public APIs with JSDoc using `@param`, `@returns`, `@example` tags
- Update JSDoc comments for any modified public APIs

### Error Handling

- Use custom error classes extending `Error` with a descriptive `name` property
- Use discriminated union types for error variants where appropriate
- `JazzError` for CoValue loading states (`UNAVAILABLE`, `DELETED`, `UNAUTHORIZED`)

### Formatting

- **Biome** formatter with **space indentation** (not tabs)
- Import organization is disabled (assist actions only)
- Pre-commit hook runs Biome check on staged files via Lefthook

## Testing

### Unit & Integration Tests (Vitest)

- **File naming**: `*.test.ts`, `*.test.tsx`
- **Location**: `src/tests/**/*.test.ts` within each package
- **Crypto**: Use `WasmCrypto` for test crypto implementations
- **Utilities**: `setupTestNode`, `createTestNode`, `waitFor` from test utilities
- **First run**: Execute `pnpm exec playwright install` for browser-based Vitest tests
- **Run**: `pnpm test` (watch mode) or `pnpm test --watch=false` (CI mode)

### E2E Tests (Playwright)

- **File naming**: `*.spec.ts`
- **Location**: `examples/*/tests/*.spec.ts` or `tests/*/`
- **Pattern**: Page Object Model for page interactions

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [garden-co/classic-jazz](https://github.com/garden-co/classic-jazz) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
