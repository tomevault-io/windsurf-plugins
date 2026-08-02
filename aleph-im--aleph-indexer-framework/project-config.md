---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

The Aleph Indexer Framework is a high-level abstraction for building multithreaded blockchain indexers on Aleph. It's a TypeScript monorepo managed with Lerna and npm workspaces that supports multiple blockchains (Solana, Ethereum, BSC, Oasys, Base, Avalanche).

## Common Commands

```bash
# Install dependencies (also runs build automatically via postinstall)
npm install

# Build all packages
npm run build

# Run tests
npm test

# Run tests with coverage
npm run test:cov

# Lint
npm run lint

# Lint and auto-fix
npm run lint:fix

# Run the framework locally
npm run start framework

# Run a specific indexer
npm run start <indexer-name>

# Check for circular dependencies
npm run deps:fix

# Generate documentation
npm run docs

# Release new version (uses lerna)
npm run version

# Publish to npm (uses lerna)
npm run publish
```

## Publishing with OIDC Trusted Publishing

This project uses **OIDC trusted publishing** (available in Lerna v9+) for secure package publishing to npm without using long-lived tokens.

### How It Works

- Publishing happens automatically via GitHub Actions (`.github/workflows/publish.yml`) when tags are pushed
- Uses OpenID Connect (OIDC) tokens instead of traditional `NPM_TOKEN` secrets
- Generates signed provenance attestations for published packages
- Each package must be configured on npm to require publishing from GitHub Actions

### GitHub Actions Configuration

The workflow already has the required setup:
- ✅ `id-token: write` permission for OIDC token retrieval
- ✅ `NPM_CONFIG_PROVENANCE: true` for signed attestations
- ✅ Lerna v9.0.3 with OIDC support

### npm Package Configuration (Required)

For each published package, configure trusted publishing on npm:

1. Go to `https://www.npmjs.com/package/@aleph-indexer/<package-name>/access`
2. Under "Publishing access", enable **"Require publishing from CI/CD"**
3. Select **"GitHub Actions"** as the provider
4. Add repository: `aleph-im/aleph-indexer-framework`
5. Optional: Specify workflow file or environment restrictions

### Published Packages

Configure these packages on npm:
- `@aleph-indexer/core`
- `@aleph-indexer/framework`
- `@aleph-indexer/ethereum`
- `@aleph-indexer/solana`
- `@aleph-indexer/bsc`
- `@aleph-indexer/oasys`
- `@aleph-indexer/oasys-verse`
- `@aleph-indexer/base`
- `@aleph-indexer/avalanche`

### Triggering a Release

```bash
# Create and push a version tag
npm run version  # Interactive version bump
git push && git push --tags

# Or manually tag
git tag v1.7.2
git push origin v1.7.2
```

The GitHub Actions workflow will automatically publish with OIDC authentication.

### ⚠️ Lockfile: do NOT move `encoding` out of `dependencies`

The root `package.json` declares `encoding` under **`dependencies`** (not
`optionalDependencies`) on purpose. `node-fetch` pulls `encoding` as an
optional dependency, and npm resolves optional dependencies differently per
platform: on **macOS** `npm install` prunes `encoding`/`iconv-lite` from
`package-lock.json`, while **Linux CI** requires them — so the moment a macOS
dev runs `npm install`, the lockfile drifts and `npm ci` fails in CI with
`Missing: encoding ... from lock file`.

Keeping `encoding` as a regular dependency forces npm to lock it on every
platform. **Do not** revert it to `optionalDependencies`, and after any
`npm install`/`lerna version` on macOS, verify `encoding` and
`iconv-lite@0.6.3` are still present in `package-lock.json` before committing.
CI also pins Node to `24` (matching the local npm major) for the same reason.
A CI guard fails the build if `encoding` leaves `dependencies`.

## Architecture

### Monorepo Structure

The repo contains these packages in `packages/`:
- **core**: Base utilities - storage (LevelDB), GraphQL API server, rate limiting, configuration
- **framework**: Main SDK and three core services (Fetcher, Parser, Indexer) with Moleculer for inter-service communication
- **ethereum**, **solana**, **bsc**, **oasys**, **oasys-verse**, **base**, **avalanche**: Blockchain-specific implementations
- **indexer-example**: Reference implementation showing how to build a custom indexer

### Three-Service Architecture

All services communicate via Moleculer (microservices framework) using configurable transports (Thread, TCP, NATS):

1. **Fetcher** (`packages/framework/src/services/fetcher/`): Tracks accounts and fetches raw transactions. Stores backup state for crash recovery. Sends raw data to Parser.

2. **Parser** (`packages/framework/src/services/parser/`): Receives raw transactions from Fetcher, splits them into instructions, extracts participating accounts.

3. **Indexer** (`packages/framework/src/services/indexer/`): Intermediary storage for backup. Maintains communication channels with Fetchers and Parsers via event queues. Tracks fetching progress per account.

### Building a Custom Indexer

Developers extend the framework by implementing:
- **MainDomain** (extends `IndexerMainDomain`): Singleton that discovers accounts to index via `discoverAccounts()` method

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aleph-im/aleph-indexer-framework](https://github.com/aleph-im/aleph-indexer-framework) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
