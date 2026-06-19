---
trigger: always_on
description: **dot-console** is a Substrate development console for the Polkadot ecosystem. It's an interactive web application that allows developers to:
---

# Dot-Console: Agent Instructions

## Project Overview

**dot-console** is a Substrate development console for the Polkadot ecosystem. It's an interactive web application that allows developers to:

- Explore on-chain data (blocks, extrinsics, events, storage)
- Query runtime constants, storage entries, and runtime APIs
- Interact with multiple Substrate-based networks (Polkadot, Kusama, Paseo, Westend, and their parachains)
- Manage accounts and wallets with support for multiple wallet providers (Ledger, Vault, WalletConnect, Mimir)
- View decoded codec data with a rich UI

## Tech Stack

| Layer                | Technology                                       |
| -------------------- | ------------------------------------------------ |
| **UI Framework**     | React 19.2.5 (Strict Mode enabled)               |
| **Routing**          | TanStack Router 1.168.22 (file-based, type-safe) |
| **State Management** | Jotai 2.19.1 (primitive atoms)                   |
| **Blockchain Data**  | @reactive-dot/core + @polkadot-api/view-builder  |
| **Styling**          | PandaCSS 1.9.1 + Park UI 0.43.1 (utility CSS)    |
| **UI Components**    | Ark UI 5.34.1 (headless components)              |
| **Build Tool**       | Vite 8.0.8 with React Compiler                   |
| **Language**         | TypeScript 6.0.2 (strict config)                 |
| **Code Quality**     | ESLint (React-specific rules), Prettier          |

## Architecture

```
src/
├── routes/           → File-based TanStack Router (auto code-splitting)
│   ├── __root.tsx    (providers, root layout, BlockTracker setup)
│   ├── _layout.tsx   (app chrome: top bar, navigation, chain selector)
│   └── _layout/      (feature routes)
│
├── features/         → Domain-driven feature modules (explorer, governance, staking, etc.)
│   └── {feature}/
│       ├── components/
│       ├── stores/    (Jotai atoms scoped to feature)
│       └── types.ts
│
├── components/       → Reusable UI layer
│   ├── ui/           (40+ generated styled-system components)
│   ├── param/        (codec renderers for blockchain types)
│   ├── *-form.tsx    (query builders with Zod validation)
│   └── query-result.tsx (async query execution template)
│
├── hooks/            → Custom React hooks
│   ├── chain.ts      (chain navigation: relay, people chains)
│   ├── metadata.ts   (metadata loading with v14/v15/v16 fallback)
│   ├── view-builder.ts (query building helpers)
│   └── use-*.ts      (routing, state helpers)
│
├── config.ts         → Reactive-Dot config (chains, providers, wallets)
├── types.ts          → Query type definitions
└── utils.ts          → Byte conversion, memoization, etc.
```

## Development Workflow

### Start Development

```bash
yarn dev     # Vite dev server with HMR on port 5173
```

### Build & Production

```bash
yarn build   # Production build
yarn preview # Preview built app
```

### Quality Checks

```bash
yarn lint    # ESLint check
```

### Post-Install

```bash
yarn postinstall  # Auto-runs: papi (generates API types) + panda codegen (CSS)
```

## Key Conventions

### Naming

- **Files**: kebab-case (e.g., `account-select.tsx`, `metadata.ts`)
- **Components**: PascalCase (e.g., `AccountSelect`, `QueryResult`)
- **Hooks**: `use*` prefix (e.g., `useChainId()`, `useLazyLoadQuery()`)
- **Jotai Atoms**: `*Atom` suffix (e.g., `blockMapAtom`, `selectedAccountAtom`)
- **Callbacks**: `on*` prefix (e.g., `onChangeAccount`)
- **Type Imports**: Always use `import type { ... }` (ESLint enforced)
- **Unused Variables**: Prefix with `_` (e.g., `_unused`, `_error`)

### Component Patterns

**Controlled + Uncontrolled Pattern**: Many components support both modes using discriminated unions:

```typescript
export type MyComponentProps =
  | ControlledProps // value + onChange (parent manages state)
  | UnControlledProps; // defaultValue (component manages state)
```

### Query Building Pattern

1. Define query structure in [src/types.ts](src/types.ts)
2. Create form builder component with Zod validation
3. Execute query with `useLazyLoadQuery()` inside Suspense
4. Wrap with `ErrorBoundary` for error handling

See [src/components/query-result.tsx](src/components/query-result.tsx) for the standard template.

### State Management

- Use Jotai atoms for global or feature-scoped state
- Define atoms in feature `stores/` directories
- Import atoms for derived state and custom read/write functions
- See [src/features/explorer/stores/blocks.ts](src/features/explorer/stores/blocks.ts) for garbage collection pattern

### Styling

```typescript
import { css } from "styled-system/css"
import { box, flex, grid } from "styled-system/jsx"

// Utility classes via css()
<div className={css({ color: 'red.500', padding: '2' })}>

// JSX factory components (preferred)
<Flex direction="column" gap="4">
```

All colors and tokens defined in [panda.config.ts](panda.config.ts).

### Chain Navigation

```typescript
import {
  getRelayChainId,
  usePeopleChainId,
  useRelayChainId,
} from "~/hooks/chain";

// Derive relay chain from parachain
getRelayChainId("kusama_asset_hub"); // → 'kusama'

// Get people chain for current relay
const peopleChainId = usePeopleChainId();
```

## Critical Files & Patterns


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [buffed-labs/dot-console](https://github.com/buffed-labs/dot-console) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-19 -->
