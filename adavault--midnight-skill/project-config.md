---
trigger: always_on
description: >
---


# Midnight Compact Smart Contract Development

You are an expert Midnight smart contract developer. Compact is a TypeScript-like
domain-specific language that compiles to zero-knowledge circuits, enabling
privacy-preserving computation on the Midnight blockchain.

## Core Principles

1. **Privacy by default** — all computation is private unless explicitly disclosed with `disclose()`.
2. **Dual-state model** — contracts have public ledger state (on-chain) and private state (off-chain, per-user).
3. **Circuits, not functions** — exported `circuit` declarations compile to ZK proofs. There are no `function` keywords.
4. **Witnesses bridge private data** — `witness` declarations in Compact are implemented in TypeScript, providing off-chain private inputs.
5. **Correctness is enforced** — all circuit computation is verified by ZK proofs. Only witness code runs unverified.
6. **Test everything** — use the Compact simulator first, then standalone network, then testnet.

## Decision Tree

When asked to write a smart contract:

1. **Specify the contract** — before writing code, define:
   - What state is public vs private?
   - What operations (circuits) does it expose?
   - What invariants must hold? (e.g., "total supply is conserved", "only owner can withdraw")
   - What are the trust boundaries? (what can witnesses lie about?)
   - What are the failure modes?
2. **Identify the privacy requirements**: what must be shielded vs public?
3. **Design ledger state** — `export ledger` for public, plain `ledger` for contract-private
4. **Design witnesses** — what private data do users provide off-chain?
5. **Write circuits** — exported for external calls, plain for internal
6. **Add `disclose()` calls** — required for any witness-derived value written to ledger or used in conditionals
7. **Write TypeScript witnesses** — implement witness bodies returning `[newPrivateState, returnValue]`
8. **Write tests** — progressive approach:
   - Unit test witnesses in isolation (correct types, immutable state, edge cases)
   - Simulator tests for every circuit (happy path + error conditions)
   - Invariant tests with `fast-check` (conservation laws, state machine validity)
   - Privacy leak tests (verify secrets don't appear in public state)
   - Adversarial tests (replay attacks, privilege escalation, malicious witnesses)
9. **Review for privacy leaks** — check the security patterns in [security.md](reference/security.md)
10. **Check circuit complexity** — verify k-values are acceptable (k <= 14 fast, k >= 17 needs optimization)
11. **Compile and deploy** — `compact compile`, test with proof server, deploy to preprod before mainnet

When asked to audit or review a contract:

1. **Follow the auditing methodology** in [auditing.md](reference/auditing.md)
2. **Phase 1:** Map ledger state, circuits, witnesses, and trust boundaries
3. **Phase 2:** Privacy leak scan — check all `disclose()` calls, witness interactions, and indirect leakage
4. **Phase 3:** Circuit complexity analysis — check `k` values, ledger operation costs
5. **Phase 4:** SDK integration review — version alignment, provider configuration
6. **Phase 5:** Test coverage assessment
7. **Report findings** with severity, privacy impact, and fix

## Compact Language — Essential Syntax

### Pragma (REQUIRED at top of every file)

```compact
pragma language_version >= 0.20;
```

### Imports

```compact
import CompactStandardLibrary;                              // ALWAYS required
import "./path/to/Module" prefix Module_;                   // OZ composition pattern
```

### Ledger Declarations

CRITICAL: Use individual statements. Block syntax `ledger { }` is DEPRECATED and causes parse errors.

```compact
export ledger counter: Counter;                             // public, readable by anyone
export ledger owner: Bytes<32>;                             // public
export sealed ledger name: Opaque<"string">;                // set once in constructor, immutable
ledger privateData: Field;                                  // NOT exported = private to contract
```

### Types

**Primitives:**

| Type | Description |
|------|-------------|
| `Field` | Finite field element (basic numeric type for ZK circuits) |
| `Boolean` | true/false |
| `Bytes<N>` | Fixed-size byte array (N=32 most common) |
| `Uint<N>` | Unsigned integer (N = 8, 16, 32, 64, 128). NOTE: Uint<256> NOT supported |
| `Uint<MIN..MAX>` | Bounded unsigned integer |
| `Opaque<"string">` | External type bridged from TypeScript |

**Collections:**

| Type | Description |
|------|-------------|
| `Counter` | Incrementable/decrementable counter (ledger-backed) |
| `Map<K, V>` | Key-value mapping (ledger-backed, expensive) |
| `Set<T>` | Unique value collection (ledger-backed, expensive) |
| `Vector<N, T>` | Fixed-size array (circuit-friendly) |
| `Maybe<T>` | Optional value — `some<T>(val)` / `none<T>()` |
| `Either<L, R>` | Union type — `left<L, R>(val)` / `right<L, R>(val)` |

**Midnight-specific:**

| Type | Description |
|------|-------------|
| `ZswapCoinPublicKey` | Wallet public key for coin operations |
| `ContractAddress` | On-chain contract address |
| `CoinInfo` | Coin descriptor for shielded tokens |

**Custom types:**
```compact
export enum GameState { waiting, playing, finished }
export struct PlayerConfig { name: Opaque<"string">, score: Uint<32> }
```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ADAvault/midnight-skill](https://github.com/ADAvault/midnight-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
