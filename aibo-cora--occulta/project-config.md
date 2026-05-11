---
trigger: always_on
description: **Don't assume. Don't hide confusion. Surface tradeoffs.**
---

# CLAUDE.md

## 1. Think Before Coding

**Don't assume. Don't hide confusion. Surface tradeoffs.**

Before implementing:
- State your assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them - don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.
- Group related properties into a single unit.

## 2. Simplicity First

**Minimum code that solves the problem. Nothing speculative.**

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- If you write 200 lines and it could be 50, rewrite it.

Ask yourself: "Would a senior engineer say this is overcomplicated?" If yes, simplify.

## 3. Surgical Changes

**Touch only what you must. Clean up only your own mess.**

When editing existing code:
- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- If you notice unrelated dead code, mention it - don't delete it.

When your changes create orphans:
- Remove imports/variables/functions that YOUR changes made unused.
- Don't remove pre-existing dead code unless asked.

The test: Every changed line should trace directly to the user's request.

## 4. Goal-Driven Execution

**Define success criteria. Loop until verified.**

Transform tasks into verifiable goals:
- "Add validation" → "Write tests for invalid inputs, then make them pass"
- "Fix the bug" → "Write a test that reproduces it, then make it pass"
- "Refactor X" → "Ensure tests pass before and after"

For multi-step tasks, state a brief plan:
```
1. [Step] → verify: [check]
2. [Step] → verify: [check]
3. [Step] → verify: [check]
```

---

### Reasoning

Always use effort=max, unless specified overwise.

## Build & Test

This is a native Xcode project with no external package manager (no CocoaPods, SPM, or Carthage).

- **Open:** `open Occulta.xcodeproj`
- **Build/Run:** Cmd+R in Xcode, targeting a physical iPhone 11+ (U1 chip required for NearbyInteraction)
- **Test all:** Cmd+U in Xcode, or via CLI:
  ```
  xcodebuild test -project Occulta.xcodeproj -scheme OccultaTests -sdk iphonesimulator
  ```
- **Run single test:** Use Xcode's test navigator diamond button, or filter by class:
  ```
  xcodebuild test -project Occulta.xcodeproj -scheme OccultaTests -sdk iphonesimulator -only-testing:OccultaTests/CryptoForwardSecrecyTests
  ```
- **Build only (simulator):**
  ```
  xcodebuild build -project Occulta.xcodeproj -scheme Occulta -destination 'generic/platform=iOS Simulator'
  ```

**Requirements:** iOS 16.0+, Xcode 16+. Physical device needed for Secure Enclave and NearbyInteraction; unit tests use `TestKeyManager` to bypass SE.

## Architecture

Occulta is an offline-first, serverless iOS contact book where physical proximity (UWB/Bluetooth) serves as the key distribution mechanism. All cryptography uses Apple frameworks only (CryptoKit + Security.framework).

Privacy and Security are paramount. There can be no vulnerabilities. Consider all possible attack vectors.

### Layers

**Crypto layer** — pure functions, no side effects, fully testable via `KeyManagerProtocol`:
- `Manager.Key` — P-256 Secure Enclave operations (create, derive, export)
- `Manager.Crypto` — AES-256-GCM encryption/decryption, ECDSA signing
- `PrekeyManager` — SE prekey batch generation and consumption (forward secrecy)
- `Crypto+Manager+ForwardSecrecy` — `seal()` / `open()` for the v3fs protocol

**Manager/Service layer** — stateful, `@Observable`, SwiftData-backed:
- `ContactManager` — SwiftData CRUD; all contact fields encrypted before storage
- `ExchangeManager` — MultipeerConnectivity + NearbyInteraction orchestration
- `PassphraseManager` — contact export/import encryption
- `PortingManager` — device migration
- `IdentityChallenge.Manager` — three-phase identity verification lifecycle (create / respond / verify)
- `IdentityChallenge.Coordinator` — `@Observable` bridge between the Manager and SwiftUI sheets

**UI layer** — SwiftUI, tab-based. Views have no direct crypto knowledge; they go through managers.

### Cryptographic Protocol

**Identity key:** P-256 in Secure Enclave, tag `"master.key.privacy.turtles.are.cute"`, exported as X9.63 uncompressed (65 bytes).

**Session key derivation:** `ECDH(ourSEKey, peerPub)` → raw 32-byte secret → `HKDF-SHA256(salt: XOR(peerPub, ourPub), info: contextString)` → 32-byte AES key. When a contact has ML-KEM quantum material (all UWB-exchanged contacts do), the session key is hybrid: P-256 shared secret XOR-combined with both ML-KEM secrets before HKDF. `QuantumKeyMaterial` is stored encrypted per contact key record and threaded through all seal/open operations including identity challenges.

**Encryption:** AES-256-GCM, 96-bit random nonce per message, AAD = version ∥ sorted SecrecyContext fields. Wire format: `nonce ∥ ciphertext ∥ tag` (CryptoKit `.combined`).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aibo-cora/occulta](https://github.com/aibo-cora/occulta) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-01 -->
