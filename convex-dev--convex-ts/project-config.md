---
trigger: always_on
description: **convex.ts** is the official TypeScript/JavaScript client library for the [Convex decentralized lattice network](https://convex.world). This is a monorepo containing client libraries and demo applications for interacting with the Convex DLT platform.
---

# Claude Code Guidelines for convex.ts

## Project Overview

**convex.ts** is the official TypeScript/JavaScript client library for the [Convex decentralized lattice network](https://convex.world). This is a monorepo containing client libraries and demo applications for interacting with the Convex DLT platform.

### Vision

Convex is building fair, inclusive, efficient, and sustainable economic systems based on decentralized technology. The convex.ts library provides developers with idiomatic TypeScript/JavaScript APIs to interact with the Convex network - creating accounts, submitting transactions, querying state, and managing cryptographic keys.

---

## Repository Structure

This is a **pnpm workspace monorepo** containing multiple packages:

```
convex.ts/
├── packages/
│   ├── convex-client/        # Main TypeScript client (@convex-world/convex-ts)
│   ├── convex-react/         # React hooks and components
│   └── demo-site/            # Next.js demo application
├── CLAUDE.md                 # This file
├── README.md                 # User-facing documentation
├── DEPLOY.md                 # Publishing guide
├── package.json              # Monorepo root configuration
└── pnpm-workspace.yaml       # Workspace configuration
```

### Package Details

| Package | npm Name | Purpose | Status |
|---------|----------|---------|--------|
| `convex-client` | `@convex-world/convex-ts` | Core client library | Ready for npm release |
| `convex-react` | `@convex-world/convex-react` | React integration | Development |
| `demo-site` | `@convex-world/demo-site` | Demo application | Development |

---

## Key Technologies

- **TypeScript 5.x** - Type-safe development
- **pnpm** - Fast, disk-efficient package manager with workspace support
- **Node.js 18+** - Runtime environment
- **axios** - HTTP client for peer communication
- **@noble/ed25519** - Ed25519 cryptographic signatures
- **@noble/hashes** - Cryptographic hashing
- **Jest** - Testing framework
- **Next.js 16** - Demo site framework (demo-site package)
- **React 18/19** - UI library (convex-react, demo-site)

---

## Development Setup

### Prerequisites

- **Node.js 18+** (20+ recommended)
- **pnpm 8+** (enable with `corepack enable`)
- **Git**
- **Docker Desktop** (for running local Convex peer during testing)

### Initial Setup

```bash
# Clone the repository
git clone https://github.com/Convex-Dev/convex.ts.git
cd convex.ts

# Install dependencies (all packages)
pnpm install

# Build all packages
pnpm build
```

### Package Management

```bash
# Install dependency in specific package
pnpm --filter @convex-world/convex-ts add <package>

# Run command in specific package
pnpm --filter @convex-world/convex-ts <command>

# Run command in all packages
pnpm -r <command>
```

---

## Building

### Build All Packages

```bash
pnpm build
```

### Build Specific Package

```bash
# Build convex-client
pnpm --filter @convex-world/convex-ts build

# Build convex-react
pnpm --filter @convex-world/convex-react build

# Build demo-site
pnpm --filter @convex-world/demo-site build
```

### Clean Build

```bash
# Clean and rebuild convex-client
cd packages/convex-client
rm -rf dist
pnpm build
```

---

## Testing

Tests require a local Convex peer running in Docker.

### Start Local Convex Peer

```bash
# Start peer
docker-compose up -d

# Verify peer is running
curl http://localhost:8080/api/v1/status

# Stop peer
docker-compose down
```

### Run Tests

```bash
# Run all tests (after starting peer)
pnpm test

# Run tests for specific package
pnpm --filter @convex-world/convex-ts test

# Run with custom peer URL
CONVEX_PEER_URL=http://localhost:8080 pnpm test
```

---

## Package-Specific Guidelines

### convex-client (@convex-world/convex-ts)

**Location:** `packages/convex-client/`

The main client library. This is the primary package for npm publishing.

#### Source Structure

```
src/
├── index.ts          # Main exports
├── convex.ts         # Main Convex class
├── types.ts          # TypeScript interfaces
├── crypto.ts         # Cryptographic utilities
├── keystore.ts       # Key management
├── identicon.ts      # Address identicons
└── __tests__/        # Jest tests (excluded from build)
```

#### Key Conventions

- **Immutable by default** - Never mutate parameters or internal state
- **Async/await** - All network operations are async
- **Error handling** - Throw descriptive errors, wrap axios errors
- **TypeScript strict mode** - All code must pass strict type checking
- **ESM only** - Use `.js` extensions in imports (NodeNext module resolution)

#### Building

```bash
cd packages/convex-client
pnpm build  # Compiles src/ -> dist/
```

**Important:** The `tsconfig.json` excludes `__tests__/` to prevent test files from being compiled into `dist/`.

#### What Gets Published

Only these files are included in the npm package (see `package.json` `files` field):
- `dist/` - Compiled JavaScript and TypeScript definitions
- `CHANGELOG.md` - Version history
- `package.json` - Package metadata

Test files, source TypeScript, and development files are excluded.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Convex-Dev) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
