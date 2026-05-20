---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Solana Attestation Service (SAS) is a Solana program that enables creating, managing, and verifying digital attestations on the Solana blockchain. The system provides a framework for issuers to create schemas and issue attestations that can optionally be tokenized as SPL Token-2022 NFTs.

The repository consists of several main components:

- `program/`: Main Solana program written in Rust using Pinocchio framework
- `core/`: Shared types and utilities for schema serialization 
- `clients/`: Auto-generated client libraries for Rust and TypeScript
- `integration_tests/`: Comprehensive test suite for all program functionality
- `cereal_macro/`: Procedural macro for schema serialization

## Common Development Commands

### Building and Testing

```bash
# Build the Solana program
cargo-build-sbf

# Run integration tests
cargo-build-sbf && SBF_OUT_DIR=$(pwd)/target/sbf-solana-solana/release cargo test

# Run TypeScript client tests
cd clients/typescript && npm test
```

### IDL Generation

```bash
# Generate IDL using Shank
shank idl -r program -o idl

# Or use the npm script
pnpm run generate-idl
```

### Client Generation

```bash
# Generate Rust and TypeScript clients from IDL
pnpm run generate-clients
```

### TypeScript Client Development

```bash
# Build TypeScript client
cd clients/typescript && npm run build

# Run tests
cd clients/typescript && npm test
```

## Architecture Overview

### Core Components

The system consists of three main components:

1. **Credential**: Represents an issuer's identity and authority to create schemas and attestations
   - Contains issuer name and list of authorized signers
   - Controls who can create schemas and attestations

2. **Schema**: Defines the structure and metadata for attestations
   - Includes field definitions, layout, and validation rules
   - Can be paused/resumed and versioned
   - Supports tokenization for creating NFT attestations utilizing Token 2022

3. **Attestation**: Actual attestation data conforming to a schema
   - Contains structured data, expiry time, and nonce
   - Can be regular attestations or tokenized (as SPL tokens)
   - Can be closed by authorized signers

### Key Features

- **Schema Versioning**: Schemas can be updated while maintaining backwards compatibility
- **Tokenization**: Schemas can be tokenized to create NFT-like attestations with metadata
- **Access Control**: Multi-signature support for credential management
- **Event Logging**: Comprehensive event system for attestation lifecycle tracking
- **Expiry Management**: Attestations can have expiration dates

### Program Structure

- **program/**: Main Solana program written in Rust using Pinocchio framework
  - **src/processor/**: Business logic for each instruction with shared utilities
    - **shared/**: Common utilities (`account_checks.rs`, `pda_utils.rs`, `data_utils.rs`)
  - **src/state/**: On-chain account structures (`attestation.rs`, `credential.rs`, `schema.rs`)
  - **src/**: Core files with specific purposes:
    - `entrypoint.rs`: Program entry point and instruction discriminator routing
    - `instructions.rs`: Shank instruction definitions for IDL generation
    - `error.rs`: Custom error types and error code definitions
    - `events.rs`: Event definitions for program logging and monitoring
    - `constants.rs`: Program constants, seeds, and configuration values
    - `lib.rs`: Main library entry point and module declarations
    - `macros.rs`: Helper macros for common operations

- **core/**: Shared types and utilities for schema serialization
  - Primitive data types and schema field validation logic

- **cereal_macro/**: Procedural macros for schema serialization
  - `SchemaStructSerialize` derive macro for automatic byte representation

- **clients/**: Auto-generated client libraries for Rust and TypeScript
  - **rust/src/generated/**: Rust client (accounts, instructions, types, errors)
  - **typescript/src/generated/**: TypeScript client (accounts, instructions, programs, types)
  - **typescript/src/**: Client utilities (`pdas.ts`, `utils.ts`) and tests

- **integration_tests/**: Comprehensive test suite for all program functionality
  - **tests/helpers/**: Test utility functions and program context setup
  - **tests/**: Individual test files for each instruction type and tokenization

- **scripts/**: Build and generation scripts
  - Client generation (`generate-clients.js`) and IDL processing (`process-idl.js`)

- **examples/**: Usage examples and practical demonstrations
  - **typescript/**: TypeScript examples for credential setup, schemas, and attestations

- **idl/**: Interface Definition Language files
  - **solana_attestation_service.json**: Program IDL defining accounts, instructions, and types

### Important Constants

- Program ID: `22zoJMtdu4tQc2PzL74ZUT7FrwgB1Udec8DdW4yw4BdG`
- Token Program: `TokenzQdBNbLqP5VEhdkAS6EPFLC1PHnBqCXEpPxuEb` (Token-2022)
- Event Authority PDA: `DzSpKpST2TSyrxokMXchFz3G2yn5WEGoxzpGEUDjCX4g`

## Technology Stack

### Program Framework


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [solana-foundation/solana-attestation-service](https://github.com/solana-foundation/solana-attestation-service) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
