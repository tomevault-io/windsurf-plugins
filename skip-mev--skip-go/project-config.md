---
trigger: always_on
description: This document provides comprehensive guidelines for contributing to the Skip Go codebase, with a focus on code organization and wallet integration patterns.
---

# Skip Go Development Guidelines

This document provides comprehensive guidelines for contributing to the Skip Go codebase, with a focus on code organization and wallet integration patterns.

## Table of Contents
1. [Code Organization and Conventions](#code-organization-and-conventions)
2. [Wallet Integration](#wallet-integration)
3. [Contribution Guidance](#contribution-guidance)
4. [Storage Patterns](#storage-patterns)
5. [Performance Optimization Patterns](#performance-optimization-patterns)
6. [Error Handling Patterns](#error-handling-patterns)
7. [API Integration Patterns](#api-integration-patterns)
8. [State Management Best Practices](#state-management-best-practices)
9. [UI/UX Enhancement Patterns](#uiux-enhancement-patterns)
10. [Development Tools](#development-tools)
11. [Code Refactoring Guidelines](#code-refactoring-guidelines)
12. [Pull Request Best Practices](#pull-request-best-practices)
13. [Changeset Requirements](#changeset-requirements)
14. [Testing Standards](#testing-standards)

## Code Organization and Conventions

### Project Structure
The codebase follows a modular architecture with clear separation of concerns:

```
packages/
├── widget/src/
│   ├── components/      # Reusable UI components
│   ├── constants/       # Application constants and configurations
│   ├── hooks/          # Custom React hooks for business logic
│   ├── icons/          # Icon components
│   ├── modals/         # Modal components
│   ├── pages/          # Page-level components
│   ├── providers/      # Context providers for different chain types
│   ├── state/          # State management using Jotai atoms
│   ├── utils/          # Utility functions organized by purpose
│   └── widget/         # Main widget entry points
└── client/             # API client and chain-specific logic
```

### File and Folder Organization
- **Feature-based grouping**: Related functionality is grouped together in logical directories
- **Single responsibility**: Each file has a single, clear purpose
- **Utility separation**: Utilities are organized by domain (e.g., `fees.ts`, `crypto.ts`, `date.ts`)

### Naming Conventions
- **Components**: PascalCase (e.g., `SwapPage.tsx`, `AssetInput.tsx`)
- **Hooks**: camelCase with `use` prefix (e.g., `useCreateCosmosWallets.tsx`, `useGetBalance.ts`)
- **Constants**: SCREAMING_SNAKE_CASE for constants, camelCase for configuration objects
- **State atoms**: camelCase with `Atom` suffix (e.g., `cosmosWalletAtom`, `sourceAssetAtom`)

### Import Organization
```typescript
// 1. External dependencies
import { atom } from "jotai";
import { ChainType } from "@skip-go/client";

// 2. Internal absolute imports
import { mainnetChains } from "@/constants/chains";
import { useGetAccount } from "@/hooks/useGetAccount";

// 3. Relative imports
import { MinimalWallet } from "./types";
```

### State Management
The codebase uses Jotai for state management with a clear atom structure:
- Atoms are defined in the `state/` directory
- Each domain has its own state file (e.g., `wallets.ts`, `swapPage.ts`)
- Complex state logic is encapsulated in custom hooks

### Modular Design Patterns
- **Provider pattern**: Chain-specific providers wrap components with necessary context
- **Hook composition**: Complex functionality is built by composing smaller, focused hooks
- **Type safety**: TypeScript interfaces define clear contracts between modules

## Wallet Integration

### Architecture Overview
Wallet integration follows a modular adapter pattern that supports multiple blockchain types:

```
providers/
├── CosmosProvider.tsx    # Cosmos chain wallet provider
├── EVMProvider.tsx       # Ethereum/EVM wallet provider
└── SolanaProvider.tsx    # Solana wallet provider
```

### Wallet State Management
Centralized wallet state is managed through Jotai atoms:

```typescript
// Core wallet types
export type MinimalWallet = {
  walletName: string;
  walletPrettyName: string;
  walletChainType: ChainType;
  walletInfo: { logo?: string };
  connect: (chainId?: string) => Promise<void>;
  disconnect: () => Promise<void>;
  isWalletConnected: boolean;
  getAddress?: (props: AddressProps) => Promise<AddressResult>;
};
```

### Chain-Specific Wallet Hooks
Each chain type has dedicated hooks for wallet creation and management:
- `useCreateCosmosWallets.tsx` - Handles Cosmos ecosystem wallets
- `useCreateEvmWallets.tsx` - Manages EVM-compatible wallets
- `useCreateSolanaWallets.tsx` - Supports Solana wallet integration

### Wallet Connection Flow
1. **Provider Initialization**: Chain-specific providers wrap the application
2. **Wallet Discovery**: Available wallets are detected based on browser extensions
3. **Connection Management**: Hooks handle connection state and chain switching
4. **Address Resolution**: Unified interface for getting addresses across chain types

### Error Handling
- Asynchronous operations use try-catch blocks with specific error handling
- Wallet connection failures trigger appropriate user feedback
- Chain switching errors are handled gracefully with fallback options

### Testing Wallet Features
- Mock wallet providers for unit tests
- Integration tests verify wallet connection flows

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [skip-mev/skip-go](https://github.com/skip-mev/skip-go) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
