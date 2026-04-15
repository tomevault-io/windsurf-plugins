---
trigger: always_on
description: anonyma is a zero-dependency TypeScript library (Node ≥ 18) for PII detection and data anonymization.
---

# anonyma — Cursor Rules
# These rules are automatically applied by Cursor to every AI interaction in this repository.

## Project Summary

anonyma is a zero-dependency TypeScript library (Node ≥ 18) for PII detection and data anonymization.
It ships:
  - 27 PII detectors (email, phone, SSN, credit card, IBAN, passport, address, VIN, API keys, …)
  - 8 anonymization strategies (mask, redact, pseudonymize, hash/SHA-256, generalize, tokenize, encrypt/AES-GCM, synthesize)
  - 6 compliance presets (GDPR, HIPAA, CCPA, PCI-DSS, SOX, FERPA)
  - Reversible tokenization + LLM pipeline helpers (sanitizeForLLM / restoreFromLLM)
  - WHATWG TransformStream wrappers, batch processing, checksum validators
  - Optional Zod schemas + OpenAI/MCP tool definitions
Published as dual ESM + CJS with full TypeScript declaration files.

---

## Hard Constraints

- ZERO runtime dependencies. Never add to `dependencies` in package.json. zod is an optional peerDependency.
- No `any` type. Always use `unknown` + type guards, generics, or proper unions.
- TypeScript strict mode + `noUncheckedIndexedAccess` + `exactOptionalPropertyTypes` + `noImplicitOverride`. All code must compile clean with `tsc --noEmit`.
- Pure functions for all detectors and non-crypto strategies (no side effects, no global state mutation).
- Immutable inputs — never mutate function arguments. Always return new values.
- Use `.js` file extensions on all relative imports in source (TypeScript NodeNext module resolution).
- JSDoc/TSDoc on every exported symbol: @param, @returns, and at least one @example.

---

## File Structure

```
src/
  index.ts             ← public API barrel — only import point for end users
  types.ts             ← all TS interfaces and type aliases (zero runtime code)
  errors.ts            ← typed error hierarchy rooted at AnonymaError
  anonymize.ts         ← core engine: detect(), anonymize(), anonymizeAsync(), anonymizeObject()
  tokenize.ts          ← tokenize() / detokenize() API
  llm.ts               ← sanitizeForLLM() / restoreFromLLM()
  batch.ts             ← anonymizeBatch(), anonymizeBatchAsync(), detectBatch(), tokenizeBatch()
  presets.ts           ← GDPR/HIPAA/CCPA/PCI-DSS/SOX/FERPA PresetConfig objects
  stream.ts            ← WHATWG TransformStream wrappers (Node ≥ 18 / browser)
  crypto.ts            ← low-level Web Crypto helpers
  validators.ts        ← checksum validators: luhn, verhoeff, nhsMod11, cpfChecksum, …
  schemas.ts           ← optional Zod schemas + AI/MCP tool definitions (do not import outside this file)
  detectors/           ← one file per PII category; each exports detect*() + detect*Aggressive()
    index.ts           ← builds DETECTOR_REGISTRY and AGGRESSIVE_DETECTOR_REGISTRY
  strategies/          ← one file per anonymization strategy
    mask.ts, redact.ts, pseudonymize.ts, hash.ts, generalize.ts, encrypt.ts, synthesize.ts, tokenize.ts
tests/                 ← Vitest test suite mirroring src/
```

---

## Code Style

- `const` by default. `let` only when reassignment is needed.
- `for...of` over `.forEach()` for side-effectful loops.
- Optional chaining `?.` and nullish coalescing `??` over explicit null/undefined checks.
- Validate inputs at the top of functions and throw typed errors; avoid try/catch in hot paths.
- `readonly` on all interface properties that must not be mutated.
- Never use default parameter values that are complex objects — compute them inside the function body.
- Discriminated unions for StrategyOptions (the `strategy` string literal is the discriminant).
- `export type` for type-only exports; `as const` for literal tuples/objects.
- `satisfies` operator to validate objects against types without widening.

---

## Error Handling Rules

- All errors must extend `AnonymaError` from `src/errors.ts`.
- Call `Object.setPrototypeOf(this, new.target.prototype)` in every Error subclass constructor.
- `ValidationError(field, reason)` — for invalid arguments.
- `UnsupportedStrategyError(strategy)` — for unknown strategy names.
- `UnknownCategoryError(category)` — for unknown PII category strings.
- `CryptoNotAvailableError` — when Web Crypto API is absent.
- `EncryptionError` — when AES-GCM fails.
- `PresetNotFoundError(preset)` — when preset name is not in registry.
- Never `throw new Error(...)` directly in library code.

---

## Crypto / Security Rules

- Use Web Crypto API exclusively: `globalThis.crypto.subtle`. Never import `node:crypto`.
- Use `Uint8Array` for binary data. Never use `Buffer`.
- AES-256-GCM for encryption, PBKDF2+SHA-256 (100,000 iterations) for passphrase key derivation.
- Always generate a fresh 12-byte random IV per encrypt() call via `crypto.getRandomValues()`.
- SHA-256 with optional pepper for the hash strategy.
- async only for crypto operations, `Promise.allSettled` for batch async. All other paths must be synchronous.

---

## Adding a New Detector — Checklist

1. Create `src/detectors/<category>.ts`
   - Export `detect<Category>(text: string): PiiMatch[]`
   - Optionally export `detect<Category>Aggressive(text: string): PiiMatch[]`
   - Implement using regex exec loop (not matchAll) to control the global flag object safely
2. Add category string literal to `PiiCategory` union in `src/types.ts`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/izaccavalheiro) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
