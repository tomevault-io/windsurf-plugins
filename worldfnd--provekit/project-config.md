---
trigger: always_on
description: ProveKit is a zero-knowledge proof system toolkit by the World Foundation. It compiles Noir programs to R1CS constraints and generates/verifies WHIR proofs. The codebase is ~95% Rust with a Go recursive verifier.
---

# CLAUDE.md

## Project Overview

ProveKit is a zero-knowledge proof system toolkit by the World Foundation. It compiles Noir programs to R1CS constraints and generates/verifies WHIR proofs. The codebase is ~95% Rust with a Go recursive verifier.

## Architecture

```
Noir Circuit (.acir)
  ↓ [r1cs-compiler]
R1CS (A, B, C matrices) + Witness Builders
  ↓ [prover]
  1. Witness solving (layered: w1 → challenges → w2)
  2. R1CS compression (postcard blob, reduces peak memory)
  3. W1 commitment (Skyscraper/SHA256/Keccak/Blake3 Merkle tree)
  4. W2 commitment (if multi-challenge circuit)
  5. WHIR sumcheck
  ↓
NoirProof { public_inputs, whir_r1cs_proof }
  ↓ [verifier]
  1. Fiat-Shamir transcript replay
  2. Commitment verification
  3. Sumcheck verification
  4. Public input binding check
  ↓
Accept / Reject
```

### Crate Structure

**Core proof system (Rust):**
- `provekit/common` — Shared types: R1CS, SparseMatrix, Interner, PrefixCovector, WitnessBuilders, HashConfig, serialization
- `provekit/r1cs-compiler` — Noir ACIR → R1CS compilation with optimizations (binop batching, range check batching, spread table caching)
- `provekit/prover` — WHIR proving: witness solving, memory compression, commitment, sumcheck
- `provekit/verifier` — WHIR verification: transcript replay, sumcheck check, public input binding

**Cryptographic primitives:**
- `skyscraper/core` — Custom BN254 hash engine with SIMD-accelerated field arithmetic (aarch64). Registered globally at startup.
- `ntt` — Number Theoretic Transform for polynomial evaluation/interpolation. Supports interleaved polynomials.
- `poseidon2` — Poseidon2 hash function (BN254-specific). Used in R1CS compilation for Poseidon2 black box calls.

**Tooling:**
- `tooling/cli` — Main CLI for prove/verify commands
- `tooling/provekit-ffi` — C-compatible FFI bindings (iOS, Android, Python, Swift, Kotlin)
- `tooling/provekit-gnark` — gnark integration for Go interop
- `tooling/provekit-bench` — Benchmarking utilities
- `tooling/verifier-server` — HTTP server combining Rust API + Go verifier

**Go recursive verifier** (`recursive-verifier/`):
- Takes WHIR proof and produces Groth16 proof for on-chain verification via gnark
- CLI (`cmd/cli/`) and HTTP server (`cmd/server/`) modes
- R1CS must match the WHIR proof being verified; PK/VK must be generated together

## Critical Invariants

These invariants are critical for soundness. Violations can produce unsound proofs or verification failures.

### R1CS Constraint Satisfaction
```
For all constraints i: (A[i] · w) * (B[i] · w) = C[i] · w  (mod BN254 prime)
```
Changes to `noir_to_r1cs()`, WitnessBuilder variants, or R1CS optimization passes must preserve this.

### Fiat-Shamir Transcript Determinism
Prover and verifier must construct identical Fiat-Shamir transcripts. The domain separator is derived from the serialized `WhirR1CSScheme`. Any change to proof structure, commitment ordering, or message sequencing breaks transcript consistency and causes verification failure.

### Public Input Binding
```
public_inputs[i] == witness[1 + i]  for all i < num_public_inputs
```
The witness at position 0 is the constant `1`. `make_public_weight` and `compute_public_eval` use `n = num_public_inputs + 1` to account for this. Off-by-one here is a soundness vulnerability (see PR #321).

### Witness Layer Scheduling
Witness builders execute in layers. All builders depending on a `Challenge` must be in a later layer than the challenge source. Within a layer, execution order is irrelevant. Layers of type `Inverse` use Montgomery's batch inversion trick (single field inverse + multiplications). Violating layer ordering causes panics in `solve_witness_vec()`.

### Prover Message vs Prover Hint
- `prover_message`: Absorbed into Fiat-Shamir transcript — verifier derives challenges from it. Use for values that must be transcript-bound.
- `prover_hint_ark`: NOT absorbed into transcript — prover sends it but it doesn't affect challenges. Use only for values independently verified by WHIR (e.g., committed polynomial evaluations).

Misusing `prover_hint_ark` for a value that should be transcript-bound is a soundness vulnerability — a malicious prover can substitute arbitrary values without detection.

### NTT/Hash Engine Registration
```rust
provekit_common::register_ntt();  // Must be called once at startup
```
Registers the WHIR ArkNtt and Skyscraper hash engine globally. Forgetting this causes runtime panics.

## Key Types

### R1CS
```rust
pub struct R1CS {
    pub num_public_inputs: usize,
    pub interner: Interner,       // Deduplicates field elements in matrices
    pub a: SparseMatrix,          // Left constraint matrix
    pub b: SparseMatrix,          // Middle constraint matrix
    pub c: SparseMatrix,          // Right constraint matrix
}
```

### SparseMatrix
Uses delta-encoded column indices within rows (reduces serialized size ~30-50%). Key operations: `set()`, `iter_row()`, `transpose()`, `multiply()` (parallel via rayon).

### PrefixCovector / OffsetCovector

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [worldfnd/ProveKit](https://github.com/worldfnd/ProveKit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
