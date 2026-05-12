---
trigger: always_on
description: Development reference for Claude Code when working in this repository.
---

# CLAUDE.md — open-passkey

Development reference for Claude Code when working in this repository.

## Project Overview

open-passkey is an open-source, post-quantum-ready library for implementing passkey (WebAuthn/FIDO2) authentication. The architecture strictly separates the **Core Protocol** (raw WebAuthn cryptography and verification) from **Framework Bindings** (Go HTTP handlers, Angular components, React hooks, etc.).

## Monorepo Structure

```
open-passkey/
├── spec/vectors/              # 31 shared JSON test vectors (registration, authentication)
├── packages/
│   ├── core-go/               # Go: Core protocol (ES256, ML-DSA-65, ML-DSA-65-ES256)
│   ├── core-ts/               # TypeScript: Core protocol
│   ├── core-py/               # Python: Core protocol
│   ├── core-java/             # Java: Core protocol
│   ├── core-dotnet/           # .NET: Core protocol
│   ├── core-rust/             # Rust: Core protocol
│   ├── server-go/             # Go: HTTP handler bindings (stdlib http.HandlerFunc)
│   ├── server-ts/             # Shared TS server logic (Passkey class)
│   ├── server-{express,fastify,hono,nestjs,nextjs,nuxt,sveltekit,remix,astro}/
│   ├── server-py/             # Shared Python server logic (PasskeyHandler class)
│   ├── server-flask/          # Flask thin wrapper (src-layout: open_passkey_flask/)
│   ├── server-fastapi/        # FastAPI thin wrapper (src-layout: open_passkey_fastapi/)
│   ├── server-django/         # Django thin wrapper (src-layout: open_passkey_django/)
│   ├── server-{spring,aspnet,axum}/
│   ├── sdk-js/                # Canonical browser SDK — PasskeyClient class + IIFE bundle
│   ├── react/                 # React hooks wrapping PasskeyClient
│   ├── vue/                   # Vue composables wrapping PasskeyClient
│   ├── svelte/                # Svelte stores wrapping PasskeyClient
│   ├── solid/                 # SolidJS primitives wrapping PasskeyClient
│   ├── angular/               # Angular components + service wrapping PasskeyClient
│   └── authenticator-ts/      # Software WebAuthn authenticator for testing
├── examples/                  # Working example for every framework (23 total)
│   └── shared/                # IIFE bundle (passkey.js) + style.css for server-only examples
└── tools/vecgen/              # Go tool to generate spec/vectors/ JSON files
```

## Cryptographic Algorithms

### Supported
| Algorithm | COSE alg | COSE kty | Implementation | Notes |
|-----------|----------|----------|----------------|-------|
| ML-DSA-65-ES256 (composite) | -52 | 9 (Composite) | Go, TS, Python, Java, .NET, Rust | Hybrid PQ, draft-ietf-jose-pq-composite-sigs |
| ML-DSA-65 (Dilithium3) | -49 | 8 (MLDSA) | Go, TS, Python, Java, .NET, Rust | Post-quantum, FIPS 204 |
| ES256 (ECDSA P-256) | -7 | 2 (EC2) | Go, TS, Python, Java, .NET, Rust | Classical, all browsers support |

### Algorithm Negotiation
All server bindings send `pubKeyCredParams` with ML-DSA-65-ES256 first (preferred), ML-DSA-65 second, and ES256 third (classical fallback). The authenticator picks the first algorithm it supports. During authentication, the core libraries read the COSE `alg` field from the stored key and dispatch to the correct verifier (ES256, ML-DSA-65, or ML-DSA-65-ES256 composite).

### ML-DSA-65 COSE Key Format
```
CBOR Map {
  1 (kty): 8        // KtyMLDSA
  3 (alg): -49      // AlgMLDSA65
  -1 (pub): bytes   // Raw ML-DSA-65 public key (1952 bytes)
}
```

### ML-DSA-65-ES256 Composite COSE Key Format
Per draft-ietf-jose-pq-composite-sigs, the composite public key concatenates components:
```
CBOR Map {
  1 (kty): 9        // KtyComposite
  3 (alg): -52      // AlgCompositeMLDSA65ES256
  -1 (pub): bytes   // ML-DSA-65 public key (1952 bytes) || ECDSA P-256 uncompressed point (65 bytes)
}
```

### ML-DSA-65-ES256 Composite Signature Format
The composite signature concatenates components with a length prefix:
- `4-byte big-endian ML-DSA sig length || ML-DSA-65 sig (3309 bytes) || ES256 DER sig`
- Both components sign over the same verification data: `authData || SHA256(clientDataJSON)`
- Both must verify independently for the composite to be valid

### ML-DSA-65 Signature Verification
Unlike ES256 (which hashes then signs), ML-DSA signs the message directly:
- Verification data: `authData || SHA256(clientDataJSON)` (same as ES256)
- ML-DSA-65 signs this data directly (no additional hashing)
- Go: `cloudflare/circl/sign/mldsa/mldsa65.Verify(pubKey, message, nil, signature)`
- TypeScript: `ml_dsa65.verify(signature, message, publicKey)` from `@noble/post-quantum/ml-dsa.js` (note: signature-first argument order)

## Session Support

Opt-in **HMAC-SHA256 stateless session cookies** — not JWTs, not server-side session stores. Disabled by default (no breaking changes).

### How It Works
- Token format: `userId:expiresAtUnixMs:base64urlHmacSha256Signature`
- Cookie: `HttpOnly; Secure; SameSite=Lax; Path=/`
- Two new endpoints: `GET /session` (validate), `POST /logout` (clear cookie)
- 10-second clock skew grace period on all expiry checks
- Secret minimum: 32 characters, enforced at startup

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [locke-inc/open-passkey](https://github.com/locke-inc/open-passkey) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
