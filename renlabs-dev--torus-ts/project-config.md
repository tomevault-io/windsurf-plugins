---
trigger: always_on
description: This file provides guidance to AI assistants when working with code in this
---

# Assistant Instructions

This file provides guidance to AI assistants when working with code in this
repository.

## Project Overview

Torus TypeScript monorepo built with Turborepo and pnpm. Contains web
applications, services, and shared packages for the Torus ecosystem.

### Architecture Components

- **Apps** (`apps/`): User-facing web applications
  - `torus-allocator`: Agent weight allocation interface
  - `torus-wallet`: Token management and staking interface
  - `torus-bridge`: Cross-chain token bridge
  - `torus-governance`: DAO voting and proposals
  - `torus-portal`: Permission management and constraint creation
  - `torus-page`: Landing page
  - `torus-prophet-finder`: Prophet finder application
  - `prediction-swarm`: Prediction swarm interface
- **Services** (`services/`): Backend services
  - `torus-cache`: Caching layer for blockchain data
  - `torus-worker`: Background processing and automation
  - `swarm-twitter`: Twitter integration for swarm
  - `swarm-verifier`: Swarm verification service
  - `swarm-services`: Swarm support services
  - `swarm-filter`: Swarm filtering service
  - `swarm-api`: Prediction swarm API
- **Packages** (`packages/`): Shared libraries
  - `@torus-network/sdk`: Core Substrate/Polkadot.js integration
  - `@torus-ts/api`: tRPC API routes and database queries
  - `@torus-ts/db`: Drizzle ORM schema and database utilities
  - `@torus-ts/ui`: Shared React components and design system
  - `@torus-ts/dsl`: Domain-specific language for constraints
  - `@torus-network/torus-utils`: Utility functions and helpers
- **Tooling**: Development configuration
  - ESLint, Prettier, TypeScript, Tailwind

The SDK (`@torus-network/sdk`) is the core library for interacting with the
Torus blockchain, providing Substrate/Polkadot.js abstractions.

## Essential Commands

### Development Workflow

```sh
# Install dependencies
just install
# or: pnpm install

# Build everything
just build

# Setup database (required for Allocator and Governance apps)
just db-push

# Start development server for specific app
just dev torus-wallet
just dev torus-allocator
just dev torus-governance
just dev torus-bridge
just dev torus-page

# Watch mode development
just dev-watch torus-wallet
```

### Code Quality

```sh
# Run all checks (typecheck + lint)
just check-all

# Check specific package
just check "@torus-network/sdk"

# Fix formatting and linting
just fix

# Individual operations
just typecheck
just lint
just format
```

### Testing

```sh
# Test specific package
just test "@torus-network/torus-utils"
just test "torus-allocator"

# Test all packages
just test
```

### Database Operations

```sh
# Apply/push schema changes
just db-push
# Alternative: just db-apply

# Open database studio
just db-studio

# Export database dump
just db-dump

# Generate migration files
# Note: Migrations are handled by Atlas and stored in atlas/migrations/
```

## Development Guidelines

### Git Push Pre-Flight Check

**IMPORTANT**: Before executing `git push`, ALWAYS run `just format-fix` to ensure code formatting is correct and prevent CI/CD failures.

**Workflow when user requests git push:**

1. Run `just format-fix` to auto-fix all formatting issues
2. Stage any formatting changes if they exist
3. Proceed with `git push`

**Rationale:**

- Prevents CI/CD failures due to formatting errors
- Saves time by catching formatting issues before push
- Only needed before push, not during active development

**Example:**

```sh
# User asks to push code
just format-fix                    # Fix all formatting
git add .                          # Stage formatting fixes if any
git push origin feature-branch     # Push to remote
```

### Package Targeting

When making changes, target specific packages for faster feedback:

- Format: `just format-fix "@torus-network/torus-utils"`
- Lint: `just lint-fix "@torus-network/torus-utils"`
- Test: `just test "@torus-network/torus-utils"`

### Error Handling Priorities

1. Typecheck errors must be addressed first
2. Fix linting errors in modified packages
3. Format errors only need fixing in modified packages
4. Ignore formatting/linting errors in unmodified packages

### Package Names

Key packages use different naming patterns:

- `@torus-network/sdk` - Main SDK package
- `@torus-network/torus-utils` - Utilities package
- `@torus-ts/*` - Internal workspace packages
- Apps use simple names: `torus-wallet`, `torus-allocator`, etc.

## Project Structure Insights

### Dependencies & Build Order

- Packages must build before apps (enforced by Turborepo)
- SDK requires subspace package for type generation
- Apps depend on shared packages (@torus-ts/ui, @torus-ts/api, etc.)

### Environment Setup

- Environment variables are shared across the monorepo
- Use `.env` file in root directory
- Database connection required for Allocator and Governance apps
- Multiple blockchain environment configurations supported

### Technology Stack

- **Frontend**: Next.js with React 19, Tailwind CSS, tRPC
- **Backend**: Node.js services, Drizzle ORM, PostgreSQL
- **Blockchain**: Polkadot.js API, custom Substrate integration
- **Development**: Turborepo, pnpm workspaces, Just task runner


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [renlabs-dev/torus-ts](https://github.com/renlabs-dev/torus-ts) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-02 -->
