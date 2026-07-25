---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

### Package Management

- Use `pnpm install` (or `pnpm install --ignore-scripts` on Windows)
- Monorepo managed with Turborepo and pnpm workspaces

### Building

```bash
# Build all packages
pnpm build

# Build specific packages with dependencies
turbo run build --filter=./packages/*

# Development with watch mode
pnpm dev                    # Core thirdweb package
pnpm dashboard              # Run dashboard + dependencies
pnpm playground             # Run playground + dependencies
pnpm portal                 # Run portal docs + dependencies
```

### Testing

```bash
# Run all tests
pnpm test

# Interactive testing (thirdweb package)
cd packages/thirdweb && pnpm test:devr

# Test specific file
pnpm test:dev <path-to-test-file>

# E2E testing (dashboard)
cd apps/dashboard && pnpm playwright
```

### Code Quality

```bash
# Lint all packages
pnpm lint

# Auto-fix linting issues
pnpm fix

# Format code
turbo run format

# Biome is the primary linter/formatter
```

### Development Workflow

```bash
# Start development server for dashboard
pnpm dashboard

# Start playground for SDK testing
pnpm playground

# Generate changeset for releases
pnpm changeset

# Version packages
pnpm version-packages
```

## Repository Architecture

### Monorepo Structure

This is a Turborepo monorepo with the main thirdweb v5 SDK consolidated into `/packages/thirdweb/`. Legacy packages are in `/legacy_packages/`.

### Core Package (`/packages/thirdweb/`)

**Main Modules:**

- `client/` - ThirdwebClient foundation
- `chains/` - 50+ supported blockchain definitions
- `contract/` - Contract interaction with automatic ABI resolution
- `transaction/` - Transaction management and execution
- `wallets/` - Comprehensive wallet integration system
- `extensions/` - Modular contract extensions (ERC20, ERC721, etc.)
- `auth/` - SIWE authentication and signature verification
- `pay/` - Fiat and crypto payment infrastructure
- `storage/` - IPFS integration for decentralized storage
- `rpc/` - Low-level blockchain communication

**Exports Structure:**
The SDK uses modular exports from `src/exports/` including:

- `thirdweb.ts` - Core client and utilities
- `chains.ts` - Chain definitions
- `wallets.ts` - Wallet connectors
- `react.ts` - React hooks and components
- `extensions/` - Contract standards and protocols

### Applications (`/apps/`)

- **dashboard** - Web-based developer console (Next.js, Chakra UI)
- **playground-web** - Interactive SDK testing environment
- **portal** - Documentation site with MDX
- **nebula** - Account abstraction and smart wallet management
- **wallet-ui** - Wallet interface and testing

### Key Packages (`/packages/`)

- **thirdweb** - Main SDK (TypeScript, React, React Native)
- **engine** - thirdweb Engine API client
- **insight** - Analytics and data APIs
- **nebula** - Account abstraction client
- **service-utils** - Shared utilities across services

## Development Practices

### GitHub Workflow & Pull Requests

- **PR titles**: Must start with affected workspace in brackets (e.g. `[SDK]`, `[Dashboard]`, `[Portal]`, `[Playground]`)
- **PR descriptions**: Begin with one-sentence summary, add checklist of changes, reference issues with `Fixes #123`
- **Commits**: Keep small and topical – one logical change per commit
- **Branch naming**: Use `area/brief-topic` format (e.g. `sdk/fix-gas-estimate`). Avoid personal names
- **Reviews**: Request at least one core maintainer review. Do not self-merge unless sole package owner
- **CI requirements**: All checks (type-check, Biome, tests) must pass before merging

### Code Quality & Formatting

- **Biome**: Primary linter/formatter (rules in `biome.json`)
- **Pre-commit**: Run `pnpm biome check --apply` before committing
- **Build verification**: Run `pnpm build` after each file change to ensure everything builds
- Avoid editor-specific configs; rely on shared settings

### TypeScript Guidelines

- **Style**: Write idiomatic TypeScript with explicit function declarations and return types
- **File structure**: Limit each file to one stateless, single-responsibility function for clarity
- **Types**: Re-use shared types from `@/types` or local `types.ts` barrels
- **Interfaces vs Types**: Prefer type aliases over interface except for nominal shapes
- **Type safety**: Avoid `any` and `unknown` unless unavoidable; narrow generics when possible
- **Architecture**: Choose composition over inheritance; leverage utility types (`Partial`, `Pick`, etc.)

### Testing Strategy

- **Co-location**: Place tests alongside code: `foo.ts` ↔ `foo.test.ts`
- **Test approach**: Use real function invocations with stub data; avoid brittle mocks
- **Network mocking**: Use Mock Service Worker (MSW) for fetch/HTTP call interception
- **Test quality**: Keep tests deterministic and side-effect free
- **Running tests**: `cd packages/thirdweb && pnpm test:dev <filename>`
- **Test accounts**: Predefined accounts in `test/src/test-wallets.ts`
- **Chain forking**: Use `FORKED_ETHEREUM_CHAIN` for mainnet interactions, `ANVIL_CHAIN` for isolated tests

### SDK Development (`packages/thirdweb`)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [thirdweb-dev/js](https://github.com/thirdweb-dev/js) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
