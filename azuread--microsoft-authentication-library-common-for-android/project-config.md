---
trigger: always_on
description: These instructions guide AI (and human) PR reviews and code suggestions for the Common library (`AzureAD/microsoft-authentication-library-common-for-android`).
---

# Copilot PR Review & Domain Instructions (Common Android Auth Library)

These instructions guide AI (and human) PR reviews and code suggestions for the Common library (`AzureAD/microsoft-authentication-library-common-for-android`).  
This file is self-contained; it includes full baseline & appendices replicated from the Broker guidelines plus deeper Common-specific detail. Changes here cascade to MSAL and Broker—review with elevated rigor. ALWAYS tailor feedback to changed code.

--------------------------------------------------------------------------------

## 0. Basic Code Review Guidelines (Enforce Consistently)
- Treat each file according to its language; never mix Java and Kotlin keywords (e.g., never produce `val final`).
- Review changed code + necessary local context; do not deep-audit untouched legacy unless new change introduces or depends on a severe risk there.
- Aggregate related minor issues only when SAME contiguous snippet/function + shared remediation.
- Each comment MUST contain: Issue, Impact (why it matters), Recommendation (actionable). Provide patch suggestions for straightforward, safe fixes.
- Replacement code must compile, preserve imports/annotations/license headers, and not weaken security, nullability, synchronization.
- Do not invent unstated domain policy; if assumption needed: “Assumption: … If incorrect, disregard.”
- Do not nitpick tool-managed formatting (ktlint/Spotless/etc.).
- Avoid flagging unchanged legacy code unless the PR’s change now interacts with it in a risky way.

--------------------------------------------------------------------------------

## 1. Domain & Architecture Primer (Common Context)

### 1.1 High-Level Purpose
Common provides cross-repository primitives:
- Command architecture (TokenCommand, BrokerCommand, controllers).
- OAuth2/OIDC protocol request/response handling.
- Token cache, serialization, normalization (authority, environment, FOCI).
- Cryptography utilities (KeyStore, key wrapping, hashing, JWE/JWS support).
- Telemetry enums (SpanName, AttributeName) and instrumentation helpers.
- IPC contracts & shared data models.
- Cloud instance & regional authority discovery and validation.
- Error taxonomy & mapping.
- Utilities (Clock, RNG abstraction, JSON adapters, correlation ids).
- Potential native code for performance/secure handling.

### 1.2 Architectural Layers
1. Public Facade (parameters, builders).
2. Command Orchestrator (controller selection).
3. Controllers (cache, broker, network).
4. Protocol Layer (request construction, response parsing).
5. Cache Layer (multi-artifact atomic updates).
6. Crypto Layer (secure operations).
7. Telemetry Layer (spans, attributes, privacy classification).
8. IPC / Serialization (bundle schemas, version negotiation).
9. Utilities (time, URL, JSON).
10. Error Mapping Layer (raw → domain exceptions).

### 1.3 Command Execution Lifecycle
1. Build parameters.
2. Preflight validation (authority, scopes, claims).
3. Controller resolution (strategy chain).
4. Span start (`SpanName.CommandExecution`) with correlation_id.
5. Execution (cache check → refresh/network/broker).
6. Response integrity checks (claims presence, flags, algorithms).
7. Atomic cache write (AT/RT/ID token & metadata).
8. Result adaptation (DTO or exception).
9. Span finalization (status set, exception recorded, `span.end()` in finally).

### 1.4 Token Artifacts
- Access Token (short-lived; scope-limited).
- Refresh Token (app or family).
- ID Token (identity claims; never logged in raw form).
- Device/PRT artifacts (managed via Broker; only referenced logically).
- Derived/session keys (ephemeral cryptographic context; minimize retention).

### 1.5 Cache Model & Atomicity
- Dimensions: environment, client_id, home_account_id, tenant_id (normalized).
- FOCI fallback: family RT if app-specific RT absent.
- Atomic multi-artifact writes (avoid partial update).
- Authority canonicalization mandatory pre-keying.
- Eviction only when new artifact supersedes previous validity window.

### 1.6 Authority & Instance Discovery
- Host validated against discovery metadata.
- Regional endpoints: secure fallback path.
- Metadata caching (avoid repeated network for identical authority).
- Schema changes require migration gating.

### 1.7 IPC Schema Compatibility
- Key constants stable; additive changes maintain backward read of old keys.
- Protocol/schema/version fields preserved semantically.
- Removal/rename of existing keys without fallback = Severity: High.

### 1.8 Cryptography Details
- Approved: SHA-256/512, AES-GCM with random IV, RSA OAEP or ECDSA P-256 (where present).
- No static IV/nonce reuse; detect repeated constant patterns.
- Use `SecureRandom`.
- No plaintext private keys in SharedPreferences; KeyStore usage required.
- Key rotation atomic: old key decommission only after new key validated.

### 1.9 Telemetry Enums
- Adding attribute: uniqueness, bounded cardinality, doc comment specifying value domain & units.
- Reuse existing SpanName for similar semantics; avoid duplication.

### 1.10 Error Taxonomy
- Service: protocol-level (invalid_grant, interaction_required).
- Client: config, parsing, network unreachable.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AzureAD/microsoft-authentication-library-common-for-android](https://github.com/AzureAD/microsoft-authentication-library-common-for-android) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
