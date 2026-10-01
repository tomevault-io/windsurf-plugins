---
trigger: always_on
description: `web3p/ethereum-tx` is a PHP library for creating, parsing, signing, hashing, and serializing Ethereum transactions.
---

# ethereum-tx — Claude Code Instructions

## Project purpose

`web3p/ethereum-tx` is a PHP library for creating, parsing, signing, hashing, and serializing Ethereum transactions.

The current codebase supports:
- Legacy Ethereum transactions through `Web3p\EthereumTx\Transaction`.
- EIP-2930 typed transactions through `EIP2930Transaction` (type `0x01`).
- EIP-1559 typed transactions through `EIP1559Transaction` (type `0x02`).
- Shared typed-transaction behavior through `TypeTransaction`.
- RLP serialization, secp256k1 signing/recovery, and Keccak hashing through existing dependencies.

This is protocol/cryptography-adjacent code. Small-looking changes can change serialized bytes, transaction hashes, signatures, or address recovery.

## Repository map

- `src/Transaction.php` — legacy transaction representation, validation, RLP serialization, EIP-155 signing, hashing, and sender recovery.
- `src/TypeTransaction.php` — common implementation for EIP-2718-style typed transactions.
- `src/EIP2930Transaction.php` — EIP-2930 access-list transaction (`0x01`).
- `src/EIP1559Transaction.php` — EIP-1559 dynamic-fee transaction (`0x02`).
- `test/unit/` — PHPUnit unit tests and transaction vectors.
- `phpunit.xml` — PHPUnit configuration; unit tests are discovered from `test/unit/*Test.php`.
- `composer.json` — package/dependency and PHP compatibility constraints.

## Compatibility constraints

Read `composer.json` before proposing syntax or dependency changes.

The current package declares PHP `^7.1|^8.0`. Do not introduce syntax or APIs unavailable to the supported PHP versions unless the task explicitly changes the compatibility policy.

Do not casually upgrade or replace cryptographic, RLP, or Ethereum utility dependencies. Dependency changes require an explicit reason and compatibility review.

Preserve the public API and serialized output unless the requested change intentionally changes them.

## Working method

Before editing code:
1. Read the relevant implementation completely.
2. Read its parent/child class when inheritance is involved.
3. Search for all callers/usages of the method being changed.
4. Read the relevant tests and existing vectors.
5. Check `composer.json` and `phpunit.xml` before assuming tooling or PHP-version capabilities.
6. For protocol behavior, verify the applicable Ethereum specification or authoritative test vector instead of guessing.

When implementing:
1. Identify the behavioral invariant first.
2. Make the smallest change that satisfies it.
3. Add or update a regression test before broad refactoring.
4. Preserve byte-level output for unaffected cases.
5. Run the narrowest relevant tests, then the full suite when practical.
6. Review `git diff` before finishing and remove unrelated changes.

Do not refactor unrelated code while fixing a protocol bug.

## Tests

Install dependencies with:

    composer install

Run the configured PHPUnit suite with:

    vendor/bin/phpunit

For a focused test, use PHPUnit's filter, for example:

    vendor/bin/phpunit --filter testSign

Do not claim tests pass unless you actually ran them and saw a successful result. If the local PHP/dependency combination prevents the suite from running, report the exact blocker.

## Ethereum transaction invariants

Treat these as high-risk areas:

### Legacy transactions
- Field order is part of the RLP encoding.
- `gas` and `gasLimit` map to the same transaction field.
- EIP-155 changes signing payload and `v` handling when `chainId` is present.
- Unsigned/signing hashes and signed transaction hashes are different concepts. Preserve the existing API semantics unless intentionally changing them.

### Typed transactions
- The transaction type byte is outside the RLP payload.
- EIP-2930 uses type `0x01`.
- EIP-1559 uses type `0x02`.
- Signing/hashing must include the correct type byte and the correct subset/order of fields.
- Signature fields must not be included in the signing payload, but must be included when computing the signed transaction hash where the API requests it.
- Access lists are structured RLP data; do not flatten or stringify them casually.

### Encoding and validation
- Preserve exact handling of empty values, zero values, hexadecimal prefixes, integer values, and leading zeroes unless a test/spec proves a change is needed.
- Never use floating-point arithmetic for Ethereum integer values.
- Be careful with PHP truthiness: `0`, `'0'`, empty strings, and null can have different protocol meanings even when PHP treats some of them similarly.
- Do not normalize hexadecimal data merely for style; normalization can alter encoded bytes.

### Signing and keys
- Never print, log, commit, or persist real private keys.
- Test private keys must be fixed test vectors only.
- Do not change secp256k1/signature behavior without authoritative vectors.
- Check recovery parameter / `v` semantics separately for legacy and typed transactions.

## Review priorities

When reviewing code, prioritize:
1. Incorrect serialized bytes or field ordering.
2. Incorrect signing payload/hash.
3. Incorrect `v`, `r`, `s`, chain ID, or sender recovery behavior.
4. Incorrect zero/empty/hex handling.
5. PHP-version compatibility.
6. Public API/backward compatibility.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [web3p/ethereum-tx](https://github.com/web3p/ethereum-tx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
