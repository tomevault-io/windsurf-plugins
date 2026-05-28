---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

Morpho Lite Monorepo containing two React web applications for Morpho Blue and MetaMorpho contracts, plus a shared UIKit package. The apps are designed to work with only public RPCs—no additional infrastructure required.

**Apps:**

- **Fallback** (`apps/fallback`) - MIT licensed resilient emergency frontend with minimal dependencies
- **Lite** (`apps/lite`) - AGPL-3.0 licensed lightweight frontend for rapid multichain expansion, supports whitelabeling

**Shared Package:**

- **UIKit** (`packages/uikit`) - MIT licensed component library and utilities used by both apps

## Development Commands

### Running Apps

```bash
# Install dependencies (required first time)
pnpm install

# Development mode (builds UIKit first, then runs app)
pnpm run fallback-app:dev    # Fallback app at http://localhost:5173
pnpm run lite-app:dev        # Lite app at http://localhost:5173

# Production build
pnpm run fallback-app:build  # Builds UIKit + Fallback
pnpm run lite-app:build      # Builds UIKit + Lite

# UIKit only
pnpm run uikit:build         # Build UIKit package
pnpm run uikit:example       # Run UIKit example app
```

### Testing

```bash
# Run tests across all workspaces
pnpm run test               # Uses vitest workspace configuration

# Run tests in specific app
cd apps/fallback && pnpm run test
cd apps/lite && pnpm run test
```

Test files use pattern: `test/**/*.{test,spec}.{ts,tsx}` in each workspace.

### Code Quality

```bash
# Lint all workspaces
pnpm run lint

# Format checking/fixing (in specific workspace)
cd apps/lite && pnpm run format        # Auto-fix
cd apps/lite && pnpm run format:check  # Check only

# Type checking
cd packages/uikit && pnpm run typecheck
```

**Pre-commit Hook:** Husky runs `lint-staged` which applies ESLint and Prettier to changed files.

### Lite App Specific

```bash
cd apps/lite

# GraphQL (gql-tada for type-safe queries)
pnpm run tada:doctor        # Validate GraphQL setup
pnpm run tada:gen          # Generate GraphQL types
pnpm run tada:cache        # Cache GraphQL introspection

# Deployment
pnpm run deploy            # Build and deploy to Vercel
```

## Architecture

### Tech Stack

- **Framework:** React 19 with Vite
- **Styling:** Tailwind CSS 4 + shadcn components
- **Web3:** wagmi 2.15.2, viem 2.29.1
- **State/Caching:** @tanstack/react-query with persistence
- **Routing:** Lite app uses React Router v7 BrowserRouter
- **Type Safety:** TypeScript 5.7, strict mode enabled
- **Testing:** Vitest with jsdom, @testing-library/react

### Monorepo Structure

This is a **pnpm workspace** with three main packages:

- `apps/fallback` - Fallback app
- `apps/lite` - Lite app
- `packages/uikit` - Shared UIKit (consumed by both apps as `@morpho-org/uikit`)

**Important:** UIKit must be built before running/building apps. The dev/build commands handle this automatically.

### Key UIKit Utilities

Located in `packages/uikit/src/`:

**Hooks:**

- `use-contract-events/` - Robust `useContractEvents` hook with adaptive `eth_getLogs` fetching strategies. Automatically handles RPC constraints, retries, and parallel transport testing to find fastest RPC.
- `use-keyed-state.ts` - Prevents state desynchronization when switching chains by keying state to chain ID
- `use-debounced-memo.ts` - Memoization with debouncing
- `use-deep-memo.ts` - Deep equality memoization
- `use-request-tracking.tsx` - Tracks pending requests with context

**Utilities:**

- `lib/utils.ts` - Formatting helpers (`formatBalance`, `formatBalanceWithSymbol`, `formatLtv`, `formatApy`), token utilities, bigint comparisons
- `lib/deployments.ts` - `DEPLOYMENTS` mapping of Morpho contracts (Morpho, MetaMorphoFactory, MetaMorphoV1_1Factory) per chain with addresses and fromBlock
- `lib/chains/` - Custom chain definitions

**Lens Contracts (tevm):**

- `lens/*.sol` - Solidity lens contracts for efficient multicall data fetching
- `lens/*.ts` - TypeScript wrappers using tevm for type-safe contract interactions

**Components:**

- `components/` - shadcn-based UI components with Morpho styling

**Important:** The `restructure` function (from `@morpho-org/blue-sdk-viem`) is used in `useReadContracts` select parameters to convert tuple arrays into objects. Example in `apps/lite/src/hooks/use-markets.ts:15`.

### App-Specific Details

#### Fallback App (`apps/fallback`)

Single page app with no URL routing. Key files:

- `src/App.tsx` - Defines `wagmiConfig`, sets up providers/contexts
- `src/app/dashboard/earn-subpage.tsx` - Fetches MetaMorpho vault data via events
- `src/app/dashboard/borrow-subpage.tsx` - Fetches Morpho Blue borrow positions via events
- `src/lib/wagmi-config.ts` - Chain and RPC configuration (uses wallet RPC first, falls back to drpc.org)
- `src/lib/constants.ts` - App constants including deployment addresses

#### Lite App (`apps/lite`)

Multi-page app with React Router. Additional features:

- **GraphQL:** Uses `gql-tada` + `urql` for type-safe Merkl rewards queries
- **Whitelabeling:** Configurable via constants and curators list
- **Configuration Files:**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [morpho-org/morpho-lite-apps](https://github.com/morpho-org/morpho-lite-apps) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-28 -->
