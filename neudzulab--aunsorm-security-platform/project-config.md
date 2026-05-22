---
trigger: always_on
description: **Last Updated:** 2026-01-31
---

# Aunsorm Agent Coordination

**Version:** 0.5.0  
**Last Updated:** 2026-01-31

## Primary Directive

**All development work must align with [PROD_PLAN.md](PROD_PLAN.md).**

This repository is coordinated by specialized domain agents. All new features, refactoring, and infrastructure changes must be tracked as tasks in `PROD_PLAN.md` with checkbox format for progress tracking.

---

## Principles

- All test errors must be fixed — even warnings are unacceptable
- No deprecated dependencies are allowed
- Issues must never be suppressed; instead, they should be resolved properly
- Mocking must not be used as a replacement for actual implementations
- Sealed classes, methods, or structures must remain intact and must not be modified
- All code must compile on MSRV 1.76+
- `unsafe` code is forbidden (`#![forbid(unsafe_code)]`)

---

## Agent Responsibilities

### Crypto Agent
- **Scope:** `crates/core`, `crates/pqc`, `crates/packet`
- **Focus:** Cryptographic primitives, PQC implementations, Native RNG compliance
- **Current Priority:** Complete PQC security audit and NIST compliance validation

### Platform Agent
- **Scope:** `crates/server`, `crates/cli`, `crates/wasm`, Docker/Kubernetes manifests
- **Focus:** Microservice orchestration, API gateway, deployment automation
- **Current Priority:** Kubernetes migration and production infrastructure setup

### Identity Agent
- **Scope:** `crates/jwt`, `crates/x509`, `crates/kms`, `crates/acme`, `crates/mdm`
- **Focus:** Authentication, certificates, key management, device enrollment
- **Current Priority:** HSM integration for KMS and OAuth 2.0 complete implementation

### Interop Agent
- **Scope:** `benches/`, `fuzz/`, `tests/`, `examples/`, CI/CD pipelines
- **Focus:** Testing, benchmarking, security audits, documentation
- **Current Priority:** Achieve >80% test coverage and third-party security audit

---

## 🎲 Native RNG Mandatory (v0.4.5+)

**CRITICAL:** All cryptographic random number generation MUST use `AunsormNativeRng`.

### Forbidden Usage:
- ❌ Direct `OsRng` usage (except initial entropy seeding)
- ❌ HTTP `/random/number` endpoint calls (6.4s overhead)
- ❌ `rand::thread_rng()` or other stdlib RNGs
- ❌ `ChaCha8Rng` or other external RNG implementations (except in tests)

### Required Usage:
- ✅ `AunsormNativeRng` - Same implementation across all crates
- ✅ HKDF + NEUDZ-PCS + AACM mixing - Same algorithm as server
- ✅ 4x Performance - Native vs HTTP (1.5s vs 6.4s RSA-2048)

### Implementation Pattern:

```rust
// ✅ CORRECT - Same pattern in every crate
use crate::rng::AunsormNativeRng;

pub fn generate_key() -> Result<Key, Error> {
    let mut rng = AunsormNativeRng::new();
    Key::generate_with_rng(&mut rng)
}

// ❌ WRONG - Now forbidden
use rand_core::OsRng;

pub fn generate_key() -> Result<Key, Error> {
    let mut rng = OsRng;  // FORBIDDEN!
    Key::generate_with_rng(&mut rng)
}
```

### Crate-Specific Requirements:
- **ACME**: Ed25519, P256, RSA account keys → `AunsormNativeRng`
- **JWT**: Ed25519 signing keys, JTI generation → `AunsormNativeRng`
- **KMS**: AES-GCM nonce generation → `AunsormNativeRng`
- **X509**: RSA key generation for certificates → `AunsormNativeRng`
- **NEW CRATES**: Must create own `src/rng.rs` module

---

## Critical Rules

### 1. Code Quality Gates

Every commit must pass:

```bash
cargo fmt --all
cargo clippy --all-targets --all-features
cargo test --all-features
cargo deny check
```

### 2. Production Plan Compliance

- All new work items must be added to `PROD_PLAN.md` with `[ ]` checkbox
- Mark tasks as `[x]` only when fully completed and tested
- Do NOT modify completed tasks - create new revision tasks instead
- Each PR must reference its `PROD_PLAN.md` task

### 3. Security Requirements

- **No `unsafe` code** - `#![forbid(unsafe_code)]` enforced
- **MSRV 1.76+** - Minimum Supported Rust Version
- **Dependency audits** - `cargo audit` must be clean
- **Fuzz testing** - All parsers/decoders must have fuzz targets

### 4. Documentation Standards

- Update `README.md` for any new endpoints or services
- Update `port-map.yaml` for any port changes
- Add `CHANGELOG.md` entry for version changes
- Technical architecture changes require `PROJECT_SUMMARY.md` updates

### 5. Revision Lock Policy

Items marked as `[x]` (completed) in README, PROD_PLAN.md, or TODO.md are locked:
- Do not reopen completed items
- Create a new item with `Revize:` prefix if changes are needed
- Reference the original item in the revision

---

## JWT Response Structure - SEALED (v0.5.0+)

**⚠️ SEALED STRUCTURE - DO NOT MODIFY WITHOUT SECURITY REVIEW ⚠️**

The JWT verification response structure is **sealed** to prevent duplicate field serialization errors and maintain client compatibility.

**Canonical Structure (`crates/server/src/routes.rs`):**

```rust
#[derive(Serialize)]
pub struct JwtPayload {
    pub subject: String,
    pub audience: String,
    pub issuer: String,
    pub expiration: u64,
    #[serde(rename = "issuedAt", skip_serializing_if = "Option::is_none")]
    pub issued_at: Option<u64>,
    #[serde(rename = "notBefore", skip_serializing_if = "Option::is_none")]
    pub not_before: Option<u64>,
    #[serde(rename = "relatedId", skip_serializing_if = "Option::is_none")]
    pub related_id: Option<String>,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Neudzulab/Aunsorm-Security-Platform](https://github.com/Neudzulab/Aunsorm-Security-Platform) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
