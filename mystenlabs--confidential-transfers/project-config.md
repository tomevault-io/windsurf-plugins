---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Code Style

All code files must include this copyright header at the top:
```
// Copyright (c) Mysten Labs, Inc.
// SPDX-License-Identifier: Apache-2.0
```

### Comment Writing Guidelines

**Do NOT comment the obvious** -- comments should not simply repeat what the code does.

**When to comment**:
- Non-obvious algorithms, cryptographic constructions, or protocol details (cite the paper or spec where possible)
- Hidden invariants, preconditions, or assumptions that are not enforced by types
- Subtle security or soundness considerations (e.g., why a check is needed, what happens if it is removed)
- Workarounds for specific bugs or upstream limitations
- Temporary placeholders or stubs (mark with `TODO:`)

**When NOT to comment**:
- Self-descriptive function calls and variable assignments
- Basic control flow (if/for/while)
- Restating type signatures
- The current task, fix, or PR ("added for X", "used by Y") -- this belongs in commit messages

## Documentation

When you change code, update the surrounding documentation in the same change:
- The top-level `README.md` is the only place that explains the project (what it is, properties, issuer/user flows). Update it when public-facing behavior, properties, or flows change.
- Sub-directory READMEs (`move/README.md`, `apps/kaisho/README.md`) only cover how to build and test the code in that directory — do not duplicate project-level explanation there.
- The "Architecture" section in this `CLAUDE.md` -- update it when modules are added, removed, renamed, or change responsibility.
- Module-level doc comments in `move/sources/*.move` and TSDoc in `ts-sdk/src/*.ts` -- keep them in sync with the code below them.

If a change makes any of the above stale, fix it in the same change rather than leaving a follow-up.

## Project Overview

Confidential transactions system for the Sui blockchain enabling confidential token transfers using homomorphic encryption and zero-knowledge proofs. Two main components: a Move smart contract and a TypeScript cryptographic SDK, plus an example wallet app.

## Build & Test Commands

### Move (in `move/`)
```
sui move build          # Build the Move package
sui move test           # Run all Move tests
sui move test <filter>  # Run specific Move test by name
```

After creating or editing any `.move` file, format it before pushing:
```
npx @mysten/prettier-plugin-move -w <file>   # or -c to check
```
CI's "Check Move formatting" job runs this plugin over every `.move` file (excluding
`build/`), and `sui move build` does **not** catch formatting issues — so a build-clean
file can still fail CI.

### WASM bindings (in `utils/bulletproofs-wasm/`)
```
pnpm build:wasm         # Build both wasm-pack targets (nodejs/ + web/)
```
`@contra/bulletproofs-wasm` wraps `fastcrypto::bulletproofs` and is consumed by
`ts-sdk` via a `file:` dependency. Requires the Rust toolchain with the
`wasm32-unknown-unknown` target plus `wasm-pack`.

### TypeScript SDK (in `ts-sdk/`)
```
pnpm install            # Install dependencies
pnpm build              # Type-check + bundle (tsdown)
pnpm test               # Run all unit tests (vitest)
pnpm vitest <filter>    # Run specific test by name/path
```

## Important: Build the WASM bindings before ts-sdk

`ts-sdk` depends on `@contra/bulletproofs-wasm` (`file:../utils/bulletproofs-wasm`).
pnpm packs `file:` deps at install time, so run `pnpm build:wasm` in
`utils/bulletproofs-wasm/` *before* `pnpm install` in `ts-sdk/`. If you change the
Rust crate, rebuild the package and re-run `pnpm install --force` in `ts-sdk/`
so the freshly built `nodejs/` + `web/` outputs are re-packed.

## Important: Rebuild ts-sdk after changes

The app (`apps/kaisho`) consumes `ts-sdk` from its built `dist/` output, not the source. After any change to `ts-sdk/src/`, always run `pnpm build` in `ts-sdk/` before testing in the app. A stale dist will silently use old code and cause hard-to-diagnose runtime errors.

## Important: Recompile Move bytecodes for the kaisho app after Move changes

The kaisho app publishes contracts from a pre-compiled bytecode bundle at `apps/kaisho/public/bu_token_bytecodes.json`, which contains both the BU test token (`apps/kaisho/move/bu_token`) and the bundled `contra` modules (`move/sources`). After any change to either Move package, run `pnpm compile-move` in `apps/kaisho/` and commit the updated bytecodes file. `pnpm dev` recompiles automatically; `pnpm build` (and Vercel) does not.

## Architecture

### Cryptographic Foundation
- **Ristretto255** group throughout, with two generators: `g` (standard) and `h` (hash-to-curve derived, unknown discrete log relationship to `g`)
- **Twisted ElGamal** encryption with message-in-exponent: ciphertext `(c = r*g + m*h, d = r*pk)`, supporting homomorphic add/subtract
- **Pedersen commitments**: `commit = m*h + blinding*g`, additively homomorphic
- **U64 amounts encoded as four u16 limbs** to prevent overflow when adding encrypted values; an `EncryptedBalance<T>` tracks a count of merged u16-bounded values that bounds limb growth so it stays decryptable

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MystenLabs/confidential-transfers](https://github.com/MystenLabs/confidential-transfers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-09 -->
