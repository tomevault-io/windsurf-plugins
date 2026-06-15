---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Important: Pull Request Workflow

**ALWAYS submit changes via pull requests (PRs) - never commit directly to main.**

### Git Shortcuts (defined in `.github/.gitconfig`)

This repository has custom git aliases configured:
- **`git dev <branch-name>`** - Create and switch to `dev/$user/$branch-name`, auto-synced with origin/main
- **`git sync`** - Rebase current branch with origin/main
- **`git pr <number>`** - Checkout a PR locally to `pr/$number`
- **`git dev-clean`** - Clean up your dev/* branches (except current)
- **`git pr-clean`** - Clean up all pr/* branches

### Recommended Workflow

1. Create a dev branch: `git dev my-feature`
2. Make your changes and commit: `git commit -m "description"`
3. Push: `git push` (auto-setup remote is enabled)
4. Create a pull request on GitHub using the printed URL
5. Wait for review and approval before merging

Direct commits to main are not allowed and may be rejected by branch protection rules.

## Architecture Overview

Sentio SDK is a TypeScript-based blockchain data indexing and analytics platform that supports multiple blockchain ecosystems. The codebase is organized as a monorepo using pnpm workspaces with the following key components:

### Core Packages
- **`packages/sdk/`**: Main SDK with blockchain-specific processors and utilities
- **`packages/cli/`**: Command-line interface for project management, deployment, and AI-powered processor generation
- **`packages/runtime/`**: Runtime engine for processing blockchain data
- **`packages/protos/`**: Protocol buffer definitions for service communication
- **`packages/action/`**: Action processing capabilities

### Blockchain Support
The SDK supports multiple blockchain ecosystems through dedicated modules in `packages/sdk/src/`:
- **Ethereum (`eth/`)**: EVM-compatible chains with contract interaction, ABI decoding, and event processing
- **Aptos (`aptos/`)**: Move-based blockchain with resource and event processing
- **Sui (`sui/`)**: Move-based blockchain with object and transaction processing
- **IOTA (`iota/`)**: Move-based blockchain similar to Sui
- **Solana (`solana/`)**: Solana blockchain with program and instruction processing
- **Bitcoin (`btc/`)**: Bitcoin blockchain with transaction and UTXO processing
- **Cosmos (`cosmos/`)**: Cosmos SDK-based chains with transaction processing
- **Fuel (`fuel/`)**: Fuel VM blockchain with asset and transaction processing

### Key Architecture Patterns
- **Processor Pattern**: Each blockchain has dedicated processor classes that handle chain-specific data
- **Context Pattern**: Chain-specific contexts provide access to blockchain state and utilities
- **Plugin System**: Modular plugin architecture for extending functionality
- **Template System**: Code generation templates for creating type-safe contract bindings
- **Testing Framework**: Comprehensive testing utilities for all supported chains

## Development Commands

### Build Commands
```bash
# Build all packages and examples
./scripts/build-all.sh

# Build specific package
pnpm --filter "./packages/sdk" build

# Build CLI templates
pnpm --filter "./packages/cli/templates/**" build --skip-deps
```

### Test Commands
```bash
# Run all tests (builds first)
./scripts/test-all.sh

# Test specific package
pnpm --filter "./packages/sdk" test
```

### Linting and Formatting
```bash
# Lint all files
pnpm lint

# Format code
pnpm format
```

### Package Management
```bash
# Install dependencies (pnpm required)
pnpm install

# Add dependency to specific package
pnpm --filter "./packages/sdk" add dependency-name
```

## CLI Usage

The Sentio CLI (`packages/cli/`) provides project management capabilities:

### Project Management
```bash
# Create new project from template
sentio create

# Add contract ABI to project
sentio add

# Build project (generates ABIs and compiles)
sentio build

# Generate ABIs only
sentio gen

# Generate processor code using AI (run after sentio gen)
sentio generate-processor --prompt "Track token transfers and calculate volume metrics"
sentio gen-processor --prompt "Monitor staking events"  # Shorter alias

# Run tests
sentio test
```

### AI Processor Generation
```bash
# Generate processor code using Sentio AI service
# Automatically uses contract info from sentio.yaml (run after sentio gen)
sentio generate-processor --prompt "Track all token transfer events and calculate daily volume metrics"

# Override specific parameters if needed
sentio gen-processor --prompt "Monitor DEX swaps" --chain-id "1" --contract "0x123..." --project-name "dex-analytics"

# The AI will:
# - Analyze your contract ABI
# - Generate production-ready processor code
# - Include appropriate metrics and event logging
# - Follow Sentio SDK best practices
```

### Deployment
```bash
# Login to Sentio platform
sentio login --host=test

# Upload processor to platform
sentio upload --host=test

# Upload subgraph processor
sentio graph
```

## Project Structure

### Configuration Files
- **`sentio.yaml`**: Project configuration file defining project name, host, and contracts
- **`package.json`**: Standard Node.js package configuration

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sentioxyz/sentio-sdk](https://github.com/sentioxyz/sentio-sdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
