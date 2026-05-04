---
trigger: always_on
description: This document is the absolute technical "brain" for any AI agent or senior engineer working with the Aleo blockchain. It provides a granular, piece-by-piece breakdown of 80 specialized technical areas. Each section includes the **Definition**, **Leo/SDK Implementation**, **AI Interpretation Logic**, and **Critical Pitfalls**.
---

# 🧠 AGENTS.md: The Ultimate Aleo Mastery Protocol (v11.0 - Universal Error Resolution Edition)

This document is the absolute technical "brain" for any AI agent or senior engineer working with the Aleo blockchain. It provides a granular, piece-by-piece breakdown of 80 specialized technical areas. Each section includes the **Definition**, **Leo/SDK Implementation**, **AI Interpretation Logic**, and **Critical Pitfalls**.

---

## 🌎 SECTION I: Core Blockchain Architecture

### 1. Aleo Layer 1 Ledger
- **Definition**: A decentralized ledger that uses Zero-Knowledge Proofs (ZKP) to decouple computation from verification.
- **Leo Implementation**: Transitions are computed locally; only the proof is verified by the network.
- **AI Interpretation Logic**: When designing dApps, always assume user data is private by default. Every transaction is a "Shielded" operation unless specified as public.
- **Critical Pitfalls**: Don't treat Aleo like Ethereum; there is no shared "world state" that transitions can read directly. Reading on-chain data requires a specialized "Handshake" in the `finalize` block.

### 2. snarkVM Execution Engine
- **Definition**: The virtual machine that executes Aleo instructions and generates Varuna proofs.
- **AVM Implementation**: Bytecode in `.aleo` files is interpreted by snarkVM.
- **AI Interpretation Logic**: AI should focus on optimizing the "gate count" (complexity) of the code to ensure users can prove transactions on lower-end devices.
- **Critical Pitfalls**: Recursive depth limits; complex nested loops in transitions can exceed the memory limits of the prover.

### 3. Record Model (Private State)
- **Definition**: A UTXO-based state model where data is stored in encrypted "Records".
- **Leo Implementation**: `record Note { owner: address, amount: u64 }`.
- **AI Interpretation Logic**: AI must treat records as consumable objects. To "update" a balance, you must spend the old record and create a new one.
- **Critical Pitfalls**: Forgetting the `owner` field makes the record unspendable. A record can only be spent once; concurrent attempts cause UTXO contention.

### 4. Async/Finalize Separation
- **Definition**: The decoupling of private proof generation (Transition) and public state updates (Finalize).
- **Leo Implementation**: `async transition ... return finalize_call();`.
- **AI Interpretation Logic**: AI must place all global state updates (Mappings) in `finalize` blocks. The transition only "proposes" the change.
- **Critical Pitfalls**: You cannot access `mappings` inside a transition. This is the most common error for Solidity developers moving to Leo.

---

## ✍️ SECTION II: Leo Language Mastery

### 5. Strict Typing & Suffixes
- **Definition**: Every numeric literal in Leo must have an explicit bit-size and type suffix.
- **Leo Implementation**: `let x: u64 = 100u64;`, `let f: field = 12345field;`.
- **AI Interpretation Logic**: AI must reject any code missing suffixes. The compiler uses these to determine the "Gate" width in the ZK-circuit.
- **Critical Pitfalls**: Mixing `u32` and `u64` in math operations without explicit casting will fail compilation.

### 6. Inline Functions
- **Definition**: Reusable code snippets that are expanded at the call site rather than having their own proof scope.
- **Leo Implementation**: `inline function add_one(a: u64) -> u64 { return a + 1u64; }`.
- **AI Interpretation Logic**: AI should use inlines for small, non-branching math to reduce the total number of gates in a transition.
- **Critical Pitfalls**: Massive inlines called inside loops can bloat the circuit beyond the 1M gate limit.

### 7. Structs & Mappings
- **Definition**: Structs group data for transitions; Mappings store public state on the blockchain.
- **Leo Implementation**: `struct Data { id: u64 }`, `mapping counts: address => u32`.
- **AI Interpretation Logic**: AI should use structs for passing complex data between transitions and functions. Use mappings for global tallies (e.g., total votes).
- **Critical Pitfalls**: Mappings are public. Never store sensitive PII (Personally Identifiable Information) in a mapping.

---

## 🔒 SECTION III: Privacy & Cryptography

### 8. The Nullifier Pattern
- **Definition**: A technique to mark a private record as "used" on-chain without revealing which record was used.
- **Leo Implementation**: `let nullifier: field = BHP256::hash_to_field(record.nonce);`.
- **AI Interpretation Logic**: AI must implement this for any anonymous system (voting, mixer) to prevent double-spending.
- **Critical Pitfalls**: Using a non-unique input for the nullifier; this allows an attacker to "predict" and block your transaction.

### 9. Stealth Addresses
- **Definition**: One-time destination addresses derived from a recipient's public key to hide the transaction recipient.
- **Leo Implementation**: $P = H(r * A) * G + A$.
- **AI Interpretation Logic**: AI should suggest stealth addresses for dApps where "who is receiving funds" must be kept secret from chain analysis.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [thirdbase1/Crane-Signals-Guide](https://github.com/thirdbase1/Crane-Signals-Guide) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-01 -->
