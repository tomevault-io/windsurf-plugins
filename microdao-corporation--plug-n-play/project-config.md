---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Plug N Play (PNP) is a unified wallet adapter library for Internet Computer dApps supporting IC, Solana (SIWS), and Ethereum (SIWE) wallets. The project has evolved into a modular architecture with separate packages for different blockchain ecosystems.

## Project Structure

```text
w98-pnp/
   src/                      # Core IC wallet functionality
      index.ts              # Main entry point, PNP class
      config.ts             # Configuration management  
      adapters/             # IC wallet adapter implementations
         BaseAdapter.ts     # Abstract base classes
         BaseDelegationAdapter.ts
         BaseSignerAdapter.ts
         ic/                # IC-specific adapters
            IIAdapter.ts    # Internet Identity
            PlugAdapter.ts  # Plug wallet
            UnifiedSignerAdapter.ts # Signer-based wallets
      managers/             # Core functionality managers
         ActorManager.ts    # IC actor creation/caching
         ConfigManager.ts   # Configuration merging
         ConnectionManager.ts # Wallet connections
         StateManager.ts    # State machine
         ErrorManager.ts    # Error handling & logging
         StatePersistenceManager.ts # State persistence
      types/                # TypeScript definitions
         AdapterTypes.ts    # Adapter interfaces
         AdapterConfigs.ts  # Adapter configurations
         AdapterExtensions.ts # Extension system types
         SiwsAdapterInterface.ts # SIWS interface
         WalletTypes.ts     # Wallet-related types
      utils.ts              # Utility functions
      did/                  # Interface definitions
   packages/                # Modular blockchain-specific packages
      solana/              # Solana wallet adapters (SIWS)
         src/
            index.ts        # Solana extension & adapters
            SiwsAdapter.ts  # Base SIWS adapter
            SimpleSiwsAdapter.ts # Simplified implementation
            SplTokenManager.ts # SPL token operations
            extensions.ts   # Solana extension definition
      ethereum/            # Ethereum wallet adapters (SIWE) 
         src/
            index.ts        # Ethereum extension & adapters
            SiweAdapter.ts  # SIWE adapter implementation
            extensions.ts   # Ethereum extension definition
   demo/                    # Demo applications
   tests/                   # Test files
   dist/                    # Build output for main package
```

## Key Commands

### Build Commands

```bash
# Core package (IC wallets)
npm run build              # Production build with minification

# Modular packages 
npm run build:solana       # Build Solana package only
npm run build:ethereum     # Build Ethereum package only
npm run build:all          # Build all packages (main + solana + ethereum)

# Individual package builds (from package directories)
cd packages/solana && npm run build
cd packages/ethereum && npm run build
```

- Uses Vite with dynamic package selection via `BUILD_PACKAGE` environment variable
- Outputs ES module and CommonJS formats for packages
- Main package: `dist/plug-n-play.es.js` (ES module only)
- Package outputs: `packages/{solana,ethereum}/dist/`
- TypeScript declarations generated for all packages
- Assets inlined for package bundles

### Testing

```bash
npm test                   # Run tests in watch mode
npm run test:ui            # Run tests with Vitest UI
npm run coverage           # Generate test coverage report

# Package-specific testing
cd packages/solana && npm run typecheck
```

- Uses Vitest with jsdom environment
- Test files: `tests/*.test.ts`
- Coverage reports in `coverage/`
- TypeScript checking available per package

### Development

```bash
npm run dev                # Start Vite dev server (main package)
npm run preview            # Preview production build

# Package development
cd packages/solana && npm run dev

# Workspace linking for local development
npm run link:all           # Link all packages for local development  
npm run unlink:all         # Unlink packages
```

## Architecture Overview

### Modular Package System

The project uses a **monorepo with workspace packages** approach:

1. **Main Package** (`@windoge98/plug-n-play`)
   - Core IC wallet functionality (Internet Identity, Plug, NFID)
   - Base adapter classes and manager pattern
   - Configuration system with extension support

2. **Solana Package** (`@windoge98/pnp-solana`) 
   - Solana wallet adapters via Sign-In with Solana (SIWS)
   - Phantom, Solflare, WalletConnect support
   - SPL token operations and Buffer polyfills

3. **Ethereum Package** (`@windoge98/pnp-ethereum`)
   - Ethereum wallet adapters via Sign-In with Ethereum (SIWE)
   - MetaMask, WalletConnect support
   - ethers.js and viem integration

### Extension System

**Declarative Adapter Registration**: Packages export extensions that register adapters automatically.

```typescript
// Instead of manual loops
import { SolanaExtension } from '@windoge98/pnp-solana';

const pnp = createPNP({
  extensions: [SolanaExtension],  // Auto-registers Solana adapters
  adapters: {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [microdao-corporation/plug-n-play](https://github.com/microdao-corporation/plug-n-play) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-10 -->
