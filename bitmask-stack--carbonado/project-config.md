---
trigger: always_on
description: **Project:** Carbonado (bitmask-stack/carbonado)
---

# AGENTS.md — Carbonado Development Guidelines

**Project:** Carbonado (bitmask-stack/carbonado)  
**Mission:** Apocalypse-resistant archival format for consensus-critical data, with a focus on Bitcoin quantum resistance.  
**Current Status (as of 2026-07):** Symmetric v2 stack (`CARBONADO20\n`, AES-256-CTR + HMAC-SHA512 EtM) stable. **P1:** `SLICE_LEN=4096`, keyed 4 KiB Bao groups, seekable slice verify. **P2:** streaming-first encode/decode. **P3:** segment sharding. **P4:** Adamantine 1.0 directory archives (see §7.1). FEC: reed-solomon-erasure RS 4/8. Outboard + scrub complete.

**Unified streaming stack (three independent axes — do not conflate):**
| Axis | Status |
|------|--------|
| **Streaming / memory** | Phase 1 fused sync path shipped (`SeekableSpool`, streaming EtM, stripe FEC). **M1:** non-FEC verification (c6) uses `SeekWriteAt` (O(chunk) RAM); FEC verification retains O(FEC body) shard buffers under segment-wide RS geometry (`finish_into` avoids a second full logical `Vec`). Residuals: FEC O(segment body), O(sidecar) outboard verify, async encoded-body spool. See [doc/STREAMING_PARALLELISM.md](doc/STREAMING_PARALLELISM.md). **Not** the same as Bao slice/stream verification. |
| **Concurrency** | Phase 2 optional `async` / `stream_decode_async` (disk spool bridge; WASM `NotImplemented`). |
| **Parallelism** | Phase 3 `parallel` feature (default on): `std::thread::scope` RS parity; WASM serial at runtime. No rayon; Tokio is not the CPU-parallel story. |

**PQC:** `bitcoinpqc` 0.4, SLH-DSA-**SHA2**-128s sidecars only (`SLH_DSA_SHA2_128S`). Dev SHAKE-128s sidecars are incompatible — re-sign.

---

## 1. Core Principles

- **Clean cryptographic break.** This version of Carbonado does **not** contain any code to read or write v1 ECIES-encrypted files. Old encrypted archives require external migration (use an older version of the tool to extract plaintext, then re-encode with the new symmetric primitives).
- "Backward compatibility / migration path" (from the original spec) refers **only** to preserving the non-crypto properties of the format:
  - Pipeline ordering (compress(zstd-20) → encrypt → FEC → bao and reverse)
  - Flat-file portability
  - WASM/browser support
  - Bao-based streaming verification and replication proofs
  - Forward error correction (4/8 RS model — see v1-vs-v2 rationale)
  - Content addressability via the outer bao hash
- **First-principles security.** All cryptographic decisions must be justifiable from security definitions (IND-CPA for CTR, INT-CTXT for EtM, PRF properties of HMAC-SHA512, domain separation, key independence).
- **HMAC-SHA512 is mandatory** for both authentication (full 64-byte tags) and all subkey derivation. No simple splits, no SHA-256-only KDFs for key material.

---

## Critical Rules to Avoid Previous Friction

These rules were added because the same misunderstandings have caused significant frustration and wasted effort:

**Production Readiness Tracking (2026-05-30 onward)**
- All remaining work required to reach "perfect and production ready" status is tracked in a single detailed todo list managed via the todo_write tool in the current session.
- New gaps discovered during audits (AES-CTR naming, nonce documentation, WASM realities, CI strictness, unwrap sites, etc.) must be immediately added as todos.
- The list is the source of truth. Status is updated in real time as items are completed with evidence (tests passing, clippy clean, docs written, etc.).
- Never batch-close multiple items. Mark one as completed only when it is verifiably done.
- The final gate (todo 29) requires a full pass against the original spec + every rule in this file.

1. **Clean Break is Non-Negotiable**
   - This library does **not** decode v1 ECIES files. Period.
   - "Backward compatibility" or "migration path" in the original spec means preserving the non-crypto format properties (pipeline, bao, zfec, flat file, WASM, etc.). It does **not** mean the code can read old encrypted containers.
   - If old v1 data is encountered, fail with a clear message directing the user to external migration.

2. **Option Presentation Discipline**
   - When offering choices, always present **one single, clearly numbered list**.
   - Never present multiple overlapping or conflicting lists in the same response.
   - Example format:
     ```
     **Next options:**
     1. Do X
     2. Do Y
     3. Do Z
     ```
   - The user has zero tolerance for ambiguity here.

3. **The `Header` Replacement Rule** (see expanded section below)

4. **Never Claim Completion While Spec-Mandated Components Are Stubs**
   - The core of the new design consists of the symmetric primitives (AES-256-CTR + full HMAC-SHA512 EtM with domain-separated subkeys) plus **SLH-DSA sidecars via bitcoinpqc** for post-quantum signatures. SLH-DSA is asymmetric by nature and is deliberately kept outside the main (symmetric) container as sidecars only. Key derivation from passphrases (e.g. Argon2id) is the caller's responsibility before supplying a 32-byte master key.
   - Do not mark work "final", "complete", or run "end-to-end verification" passes while any of those pillars remain placeholder functions or comments only.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bitmask-stack/carbonado](https://github.com/bitmask-stack/carbonado) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
