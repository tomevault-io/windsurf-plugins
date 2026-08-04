---
trigger: always_on
description: TinyCloud is a decentralized, user-controlled cloud framework enabling data sovereignty and privacy-preserving storage. Users retain full control over their data with fine-grained access permissions through capability-based security.
---

# TinyCloud Protocol Development Guidelines

## Project Overview

TinyCloud is a decentralized, user-controlled cloud framework enabling data sovereignty and privacy-preserving storage. Users retain full control over their data with fine-grained access permissions through capability-based security.

**Core Concepts:**
- **Orbits**: User-owned data storage spaces that can be self-hosted or managed
- **Capabilities**: UCAN/CACAO-based tokens defining who can access data and how
- **DIDs**: Decentralized Identifiers for authentication without centralized authority

## Build Commands

```bash
# Build
cargo build                              # Debug build
cargo build --release                    # Production build

# Run
cargo run                                # Run locally (default port 8000)

# Test
cargo test                               # Run all tests
cargo test module_name                   # Test specific module
cargo test test_name -- --nocapture      # Single test with output

# Load Testing
k6 run --vus 10 --duration 30s test/load/k6/json_put.js
```

## Linting & Formatting

```bash
cargo clippy -- -D warnings              # Lint with warnings as errors
cargo fmt                                # Format code
cargo fmt -- --check                     # Check formatting without modifying
```

**Always run before committing:**
```bash
cargo fmt && cargo clippy -- -D warnings && cargo test
```

## Project Structure

```
tinycloud-node/
├── tinycloud-node-server/        # Main HTTP server binary (Rocket-based)
│   └── src/
│       ├── main.rs               # Server bootstrap, Prometheus metrics
│       ├── lib.rs                # Application setup, route mounting
│       ├── routes/               # API endpoint handlers
│       │   └── mod.rs            # /invoke, /delegate, /peer/generate, /healthz
│       ├── auth_guards.rs        # Request guards for authorization headers
│       ├── authorization.rs      # Auth header parsing and verification
│       ├── config.rs             # Configuration structures
│       ├── prometheus.rs         # Metrics exposition
│       ├── tracing.rs            # Distributed tracing setup
│       └── storage/              # Storage backend implementations
│
├── tinycloud-core/               # Core database layer (OrbitDatabase)
│   └── src/
│       ├── db.rs                 # Main database abstraction
│       ├── events/               # Event types (Delegation, Invocation, Revocation)
│       ├── models/               # Database entity definitions
│       ├── storage/              # Storage trait definitions and implementations
│       ├── types/                # Ability, Resource, Caveats, Metadata
│       ├── migrations/           # Database schema migrations
│       ├── hash.rs               # Content hashing (Blake2b, Blake3)
│       ├── keys.rs               # Cryptographic key management
│       └── manifest.rs           # Orbit manifest handling
│
├── tinycloud-auth/               # Shared authorization library
│   └── src/
│       ├── authorization.rs      # TinyCloudDelegation, Invocation, Revocation
│       ├── resource.rs           # TinyCloud resource URIs and paths
│       └── resolver.rs           # DID resolution
│
├── tinycloud-sdk-rs/             # Rust SDK for client applications
├── tinycloud-sdk-wasm/           # WebAssembly SDK bindings for browsers
│
├── dependencies/
│   ├── siwe/                     # EIP-4361 Sign-In with Ethereum
│   ├── siwe-recap/               # EIP-5573 SIWE ReCap capability delegation
│   └── cacao/                    # CAIP-74 Chain-Agnostic Object Capability
│
├── test/load/                    # Load testing infrastructure
│   ├── k6/                       # k6 test scripts
│   └── signer/                   # Signing utility for test capabilities
│
└── .github/workflows/            # CI/CD pipelines
```

## API Endpoints

| Method | Endpoint | Description | Auth Required |
|--------|----------|-------------|---------------|
| `POST` | `/invoke` | Execute KV operations (list, get, put, delete, metadata) | Yes |
| `POST` | `/delegate` | Create capability delegations | Yes |
| `GET` | `/peer/generate/<orbit>` | Generate orbit host key pair | No |
| `GET` | `/healthz` | Health check | No |
| `OPTIONS` | `/*` | CORS preflight | No |

**Authorization Header Format:**
```
Authorization: <base64url-encoded-UCAN-or-CACAO>
```

**KV Capabilities:**
- `kv/list` - List keys in an orbit
- `kv/get` - Read a value
- `kv/put` - Write a value
- `kv/delete` - Remove a value
- `kv/metadata` - Get value metadata

## Authentication Architecture

TinyCloud uses a three-layer capability-based authentication:

1. **UCAN (User-Controlled Authorization Network)**: JWT-like tokens encoding capabilities with delegation chains
2. **CACAO (Chain-Agnostic Capability Object)**: IPLD-encoded capabilities with SIWE signatures
3. **SIWE (Sign-In with Ethereum)**: EIP-4361 signature verification for Ethereum wallets

**Request Flow:**
```
Request → Authorization Header → Parse (UCAN/CACAO) → Verify Signature →
Validate Capability → Check Resource Permission → Execute Operation
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TinyCloudLabs/tinycloud-node](https://github.com/TinyCloudLabs/tinycloud-node) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->
