---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

JWSETKit is a Swift library for working with JSON Web Signature (JWS), JSON Web Encryption (JWE), and JSON Web Token (JWT) according to JOSE (JSON Object Signing and Encryption) standards. The library provides comprehensive cryptographic functionality for signing, encrypting, and verifying JSON web tokens with support for multiple algorithms and key types.

## Development Commands

### Building and Testing
```bash
# Build the project
swift build -v

# Run all tests
swift test -v

# Run tests with code coverage
swift test -v --enable-code-coverage

# Run tests using Make
make test
```

### Cross-Platform Testing
```bash
# Test on Linux using Docker
make linuxtest

# Clean Linux test (builds fresh container)
make cleanlinuxtest
```

### Changelog and Release
```bash
# Print a Markdown changelog from Conventional-Commit lines (last tag..HEAD by default)
make changelog
make changelog FROM=1.3.0 TO=2.1.0

# Draft/update a GitHub release with those notes (requires gh CLI)
make release VERSION=2.1.0
make release VERSION=2.1.0 DRY_RUN=1   # print notes without touching GitHub
```

### Package Management
The project uses Swift Package Manager with dependencies defined in `Package.swift`. Key dependencies:
- `swift-collections` for data structures
- `swift-asn1` for ASN.1 parsing
- `swift-crypto` for cryptographic operations
- `swift-certificates` for X.509 certificate support (`X509` trait)
- `secp256k1` for ES256K/secp256k1 support (`P256K` trait)
- `async-http-client` and `swift-http-types` for network fetching and DPoP integration (`HTTP` trait)
- `swift-testing` for testing framework

### Package Traits
Optional functionality is gated behind SwiftPM traits:
- `HTTP` (**enabled by default**): remote JWKS/`x5u`/`cnf` fetching, DPoP minting/verification on AsyncHTTPClient, NIO and swift-http-types
- `X509`: X.509 certificate support via swift-certificates
- `P256K`: secp256k1 / ES256K signature support

## Architecture Overview

### Core Components

**Base Layer** (`Sources/JWSETKit/Base/`):
- `WebContainer.swift`: Core protocol for JSON containers used throughout JOSE structures
- `ProtectedContainer.swift`: Base for protected (signed/encrypted) containers
- `Storage.swift`: Key-value storage abstraction for JOSE headers and payloads
- `Error.swift`: Centralized error handling for the library

**Entities** (`Sources/JWSETKit/Entities/`):
- **JWS** (`JWS/`): JSON Web Signature implementation with support for multiple signatures
- **JWE** (`JWE/`): JSON Web Encryption with recipient-based encryption
- **JWT** (`JWT/`): JSON Web Token built on top of JWS with registered claims, plus OIDC `at_hash`/`c_hash` computation and verification
- **SD-JWT** (`SD-JWT/`): Selective Disclosure JWT (RFC 9901) — disclosure/conceal, holder key binding via `cnf`
- **DPoP** (`DPoP/`): Demonstrating Proof of Possession (RFC 9449) — proof minting (`setDPoPProof`) and server-side verification (`verify`/`verifyBinding`) for `URLRequest`, NIO `HTTPHeaders`, and swift-http-types; replay (`jti`) tracking is the caller's responsibility
- **JOSE** (`JOSE/`): JOSE header implementations for different token types

**Cryptography** (`Sources/JWSETKit/Cryptography/`):
- **Algorithms** (`Algorithms/`): Algorithm abstractions and implementations for signature, key encryption, and content encryption
- **EC** (`EC/`): Elliptic curve cryptography (P-256, P-384, P-521, Ed25519, secp256k1/ES256K, HPKE)
- **RSA** (`RSA/`): RSA cryptography with PKCS#1 and PSS padding
- **PQC** (`PQC/`): Post-quantum cryptography (ML-DSA variants)
- **Symmetric** (`Symmetric/`): Symmetric key operations (AES, HMAC, KDF)
- **Certificate** (`Certificate/`): X.509 certificate handling and JSON Web Certificate support
- **Compression** (`Compression/`): JWE payload compression (DEFLATE via Apple Compression / zlib)
- **Hashing** (`Hashing/`): SHA-2/SHA-3 hash function identifiers and abstractions

**Network** (`Sources/JWSETKit/Network/`, requires `HTTP` trait):
- Remote fetching of JWKS (`jku`), certificates (`x5u`), and `cnf` keys over URLSession, AsyncHTTPClient, and NIO
- `JSONWebKeySetProvider` with presets (`.apple`, `.google`, `.firebase`, `.microsoft`) and `openID(_:)` discovery; no built-in caching — re-fetches on every call, caching is the host's job

### Key Design Patterns

**Generic Containers**: The library uses generic `JSONWebSignature<Payload>` and similar structures where `Payload` conforms to `ProtectedWebContainer`. This allows type-safe handling of different payload types (JWT claims, arbitrary JSON, etc.).

**Algorithm Abstraction**: Cryptographic algorithms implement the `JSONWebAlgorithm` protocol, providing consistent interfaces for signature, key encryption, and content encryption algorithms.

**Storage Pattern**: JSON data is managed through `JSONWebValueStorage` which provides dynamic member lookup and type-safe access to JOSE header fields and JWT claims.

**Cross-Platform Support**: The library uses conditional imports (`FoundationEssentials` vs `Foundation`) and platform-specific implementations to support both Apple platforms and Linux.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [amosavian/JWSETKit](https://github.com/amosavian/JWSETKit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
