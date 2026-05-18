---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## TL;DR - Development Workflow

### Branches & Commits
- **Main branch**: `main` (protected integration branch; may include audited and unaudited commits)
- **Topic branches**: `feat/*`, `fix/*`, `chore/*` (branch from `main`, PR back to `main`)
- **Hotfix branch**: `hotfix/*` (urgent fixes, branch from the currently deployed stable tag)
- **Commit format**: Use conventional commits for automatic releases
  - `feat:` → minor version bump (1.0.3 → 1.1.0)
  - `fix:` → patch version bump (1.0.3 → 1.0.4)
  - `BREAKING CHANGE:` → major version bump (1.0.3 → 2.0.0)
  - `chore:`, `docs:`, `refactor:` → patch version bump

### Publishing Flow
- **Rust crates**: Manual release process with synchronized versioning (both kora-lib and kora-cli released together)
- **TypeScript SDKs**: Changeset-based releases (require `pnpm changeset`)
- **CHANGELOG**: Auto-generated from conventional commits using git-cliff
- **GitHub releases**: Auto-generated with commit-based release notes

## Project Overview

Kora is a Solana paymaster node that provides a JSON-RPC interface for handling gasless transactions and fee abstractions. It enables developers to build applications where users can pay transaction fees in tokens other than SOL.

The repository consists of 2 main workspace crates:

- `kora-lib`: Core library with integrated RPC server functionality, signers, transaction handling, and configuration
- `kora-cli`: Unified command-line interface with RPC server and configuration commands
- `tests`: Integration tests for the entire workspace
- `sdks/`: TypeScript SDKs for client integration

## TL;DR - Authentication Methods

Kora supports two authentication methods that can be used individually or together:

1. **API Key Authentication**: Simple header-based auth using `x-api-key` header
2. **HMAC Authentication**: Request signature auth using `x-timestamp` and `x-hmac-signature` headers

Optional bot protection via **reCAPTCHA v3** can be integrated into either auth method.

**Testing:**
```bash
just integration-test           # Run all integration tests
```

## Common Development Commands

### Build & Check

```bash
# Build all workspace packages
just build

# Build specific packages
just build-lib    # Build the lib crate
just build-cli    # Build the CLI tool

# Install all binaries
just install

# Run formatter (formats and checks)
just fmt
```

### Testing

```bash
# Run unit tests
just unit-test

# Run integration tests (automatically handles environment setup)
just integration-test

# Run all tests
just test
```

#### Integration Test Environment Setup

Integration tests are fully automated using a Rust test runner binary that handles sequential test execution:

**Quick Start:**
```bash
just integration-test
```

**What happens automatically:**
1. **Solana Validator**: Starts local test validator with reset
2. **Test Environment Setup**: Creates test accounts, tokens, and ATAs
3. **Sequential Test Phases**: Runs 3 test suites with different configurations

**Test Phases (Configured in `tests/src/test_runner/test_cases.toml`):**

**Regular Tests**
- Config: `tests/src/common/fixtures/kora-test.toml` (no auth)
- Tests: Core RPC functionality, token operations, compute budget

**Auth Tests**
- Config: `tests/src/common/fixtures/auth-test.toml` (auth enabled)
- Tests: API key and HMAC authentication validation

**Payment Address Tests**
- Config: `tests/src/common/fixtures/paymaster-address-test.toml` (payment address)
- **CLI ATA Initialization**: Automatically runs `kora rpc initialize-atas` before tests
- Tests: Payment address validation and wrong destination rejection

**Multi-Signer Tests**
- Config: `tests/src/common/fixtures/signers-multi.toml`
- Tests: Multiple signer configurations

**TypeScript Tests**
- Tests: TypeScript SDK integration tests

**File Structure:**
```
tests/
├── src/
│   ├── common/
│   │   ├── fixtures/
│   │   │   ├── kora-test.toml           # Regular tests config
│   │   │   ├── auth-test.toml           # Auth tests config  
│   │   │   └── paymaster-address-test.toml  # Payment address config
│   │   ├── local-keys/
│   │   │   ├── fee-payer-local.json     # Fee payer keypair
│   │   │   ├── payment-local.json       # Payment address keypair
│   │   │   ├── sender-local.json        # Sender keypair
│   │   │   └── usdc-mint-local.json     # USDC mint keypair
│   │   └── setup.rs                     # Test environment setup
│   ├── test_runner/                     # Test runner modules
│   │   ├── accounts.rs                  # Account management
│   │   ├── commands.rs                  # Test command execution
│   │   ├── config.rs                    # Test configuration
│   │   ├── kora.rs                      # Kora server management
│   │   ├── output.rs                    # Output handling
│   │   ├── test_cases.toml              # Test phase configurations
│   │   └── validator.rs                 # Solana validator management
│   └── bin/
│       └── test_runner.rs               # Main test runner binary
├── integration/                         # Regular integration tests

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [solana-foundation/kora](https://github.com/solana-foundation/kora) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
