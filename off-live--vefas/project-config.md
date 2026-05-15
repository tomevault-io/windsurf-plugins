---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

VEFAS (Verifiable Execution Framework for Agents) is a zkTLS client that generates cryptographic proofs of HTTPS requests and responses. It enables AI agents to prove they performed external actions (e.g., "Email sent") without relying on trust, MPC, or notaries.

**Key Innovation**: Two-layer verification architecture:
1. **Layer 1 (zkVM Receipt)**: Fast cryptographic proof validation using zkVM (RISC0 or SP1)
2. **Layer 2 (TLS Validation)**: Comprehensive validation of Merkle proofs, certificates, and handshake integrity

**Supported Platforms**: RISC0 and SP1 zkVMs with CUDA acceleration support

## Build Commands

### Basic Build
```bash
# Build entire workspace
cargo build --workspace

# Build with release optimizations (recommended for proof generation)
cargo build --workspace --release
```

### Testing
```bash
# Run all tests in workspace
cargo test --workspace

# Run tests for specific crate
cargo test -p vefas-core
cargo test -p vefas-node
cargo test -p vefas-crypto

# Run integration tests (requires 180s timeout for proof generation)
cargo test -p vefas-node --test integration_tests --release --features cuda

# Run specific test
cargo test test_e2e_proof_generation_and_verification_risc0
```

### zkVM Guest Programs

**SP1 Guest Program:**
```bash
# Build SP1 guest program
cd crates/vefas-sp1/program
cargo prove build

# Or from root
cargo build -p vefas-sp1-program --release
```

**RISC0 Guest Program:**
```bash
# Build RISC0 guest program
cd crates/vefas-risc0/methods/guest
cargo build --release --target riscv32im-unknown-none-elf

# Or from root
cargo build -p vefas-risc0-methods
```

### Linting and Formatting
```bash
# Check formatting
cargo fmt -- --check

# Auto-format code
cargo fmt

# Run clippy
cargo clippy --workspace -- -D warnings
```

### Running the VEFAS Node
```bash
# Start the unified VEFAS node server (port 8080)
cargo run -p vefas-node --release

# With CUDA acceleration (requires CUDA 12, 24GB+ VRAM)
cargo run -p vefas-node --release --features cuda

# Test with curl
curl -X POST http://127.0.0.1:8080/api/v1/requests \
  -H "Content-Type: application/json" \
  -d '{"method": "GET", "url": "https://example.com", "proof_platform": "risc0"}'
```

## Architecture

### Crate Organization

**Core Infrastructure:**
- `vefas-types`: Platform-agnostic no_std types for zkTLS verification (canonical bundles, proof claims)
- `vefas-core`: Production HTTP client with TLS capture (`VefasClient`)
- `vefas-node`: Unified HTTP execution and proof verification service with REST API
- `vefas-rustls`: Custom rustls crypto provider with TLS message capture capabilities

**Cryptographic Layer:**
- `vefas-crypto`: Trait-only crate with platform-agnostic interfaces (Hash, AEAD, KDF, Signature)
- `vefas-crypto-native`: Native implementations using aws-lc-rs (host environment)
- `vefas-crypto-sp1`: SP1 zkVM implementations using SP1 precompiles
- `vefas-crypto-risc0`: RISC0 zkVM implementations using RISC0 precompiles

**zkVM Integration:**
- `vefas-sp1/`: SP1 prover (host) + guest program
  - `src/lib.rs`: `VefasSp1Prover` - proof generation/verification
  - `program/src/main.rs`: Guest program executed in SP1 zkVM
  - `script/`: Build script for compiling guest program
- `vefas-risc0/`: RISC0 prover (host) + guest program
  - `src/lib.rs`: `VefasRisc0Prover` - proof generation/verification
  - `methods/guest/src/main.rs`: Guest program executed in RISC0 zkVM

### Key Data Flow

```text
HTTP Request → VefasClient → TLS Handshake Capture → VefasCanonicalBundle + Merkle Tree
                                                              ↓
                                                    zkVM Guest Program
                                                    (SP1 or RISC0)
                                                              ↓
                                                    Cryptographic Verification
                                                              ↓
                                                    VefasProofClaim + zkVM Receipt
                                                              ↓
                                                    VerifierService (2-Layer)
                                                              ↓
                                                    ValidationResult (valid/invalid)
```

### Verification Flow (2-Layer Architecture)

**Host Proof Generation:**
1. `VefasClient` captures TLS 1.3 handshake and HTTP exchange
2. `transcript_bundle.rs` creates `VefasCanonicalBundle` with real extracted data
3. `merkle_tree.rs` builds Merkle tree for selective disclosure
4. zkVM prover generates cryptographic proof (SP1 or RISC0)
5. Returns: `{proof_data, claim, bundle}`

**VerifierService Validation:**
1. **Layer 1: zkVM Receipt Verification** (fast, cryptographic)
   - Verify proof using platform-specific prover (SP1 or RISC0)
   - Validate proof was generated by trusted zkVM program (ELF ID / VK)
   - Extract verified claim from proof

2. **Layer 2: TLS Validation** (comprehensive)
   - Validate Merkle proofs for selective disclosure fields

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Off-Live/vefas](https://github.com/Off-Live/vefas) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-08 -->
