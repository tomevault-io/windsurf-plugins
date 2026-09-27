---
trigger: always_on
description: This is the Foundry project for the on-chain standalone WHIR verifiers over the KoalaBear and BabyBear fields and the LeanVM terminal verifier. The direct WHIR paths verify only the PCS portion used by Spartan-WHIR. The LeanVM path verifies a terminal proof whose guest execution covers the complete Spartan-WHIR verifier. External source repositories and cross-crate logic anchors are linked below in [Verifier Source Anchors](#verifier-source-anchors).
---

# sol-spartan-whir -- Agent Instructions

This is the Foundry project for the on-chain standalone WHIR verifiers over the KoalaBear and BabyBear fields and the LeanVM terminal verifier. The direct WHIR paths verify only the PCS portion used by Spartan-WHIR. The LeanVM path verifies a terminal proof whose guest execution covers the complete Spartan-WHIR verifier. External source repositories and cross-crate logic anchors are linked below in [Verifier Source Anchors](#verifier-source-anchors).

## Skills

Detailed workflow guides live under `.agents/skills/*/SKILL.md`:

- `.agents/skills/forge-flamegraph-profiling/SKILL.md` — execution gas profiling with Foundry flamegraphs and `gasleft()` harness tests
- `.agents/skills/tx-gas-benchmarking/SKILL.md` — total transaction gas measurement via Anvil broadcast runs
- `.agents/skills/solidity-compiler-analysis/SKILL.md` — optimized IR, repeated computation, stack spills, and gas versus deployed bytecode
- `.agents/skills/finite-field-arithmetic-optimization/SKILL.md` — algorithm research, arithmetic bounds, and independent correctness validation
- `.agents/skills/gas-calibration-maintenance/SKILL.md` — measurement provenance, calibration refresh, source fingerprints, and schedule reports

## Protocol Compatibility Rules

Transcript byte-level compatibility between Rust and Solidity is the highest correctness risk. If the Solidity challenger produces even one different byte during observe or sample operations, every subsequent challenge diverges and the proof is rejected.

Standalone-WHIR proof data is encoded with the Rust codec/exporter conventions. The standalone-WHIR Solidity verifier has three paths:

- Native blob verifier (`*WhirBlobVerifierNative*` schedule-specific variants): production-style path. Reads the fixed-shape blob directly from calldata.
- Typed ABI verifier (`*WhirVerifier*` schedule-specific variants): parity/test path. Uses `abi.encode`/`abi.decode` for debuggability.
- Blob decode-and-delegate wrapper (`*WhirBlobVerifier*` schedule-specific variants): decodes the blob into typed structs, then delegates to the typed verifier.

The blob layout mixes encoding conventions on purpose: transcript-native little-endian sections for data fed to the challenger, plus big-endian or packed sections for Merkle/proof data. Do not reorganize it for consistency. The layout is optimized for gas, and any change needs benchmarking plus Rust fixture regeneration.

Changes to transcript ordering, proof encoding, digest layout, Merkle hashing, or domain separator construction are protocol-surface changes. State explicitly which Solidity components are affected and what needs to be regenerated or updated.

The Solidity verifier assumes Keccak hashing with domain-separation prefix bytes (`0x00` for leaves, `0x01` for nodes). The `keccak_no_prefix` feature flag in the Rust implementation must stay disabled because enabling it silently breaks Merkle verification in Solidity.

EVM verifier compatibility takes priority over Rust-only cleanliness. Reject changes that make EVM verification harder, less efficient, or incompatible with the current plan, even if they improve Rust abstraction quality or prover performance.

## Verifier Source Anchors

Use these upstream locations as the logic sources when checking Solidity behavior:

| Surface                            | Source                                                                                                                                                                                                                                                                                                             |
| ---------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| WHIR verification inside Spartan-WHIR | [spartan-whir/src/whir_pcs.rs](https://github.com/ethereum/spartan-whir/blob/main/src/whir_pcs.rs), [whir-p3/src/whir/verifier/mod.rs](https://github.com/alxkzmn/whir-p3/blob/csp/src/whir/verifier/mod.rs), and [whir-p3/src/whir/verifier/sumcheck.rs](https://github.com/alxkzmn/whir-p3/blob/csp/src/whir/verifier/sumcheck.rs) |
| Full Spartan IOP reference only     | [spartan-whir/src/protocol.rs](https://github.com/ethereum/spartan-whir/blob/main/src/protocol.rs) and [spartan-whir/src/sumcheck.rs](https://github.com/ethereum/spartan-whir/blob/main/src/sumcheck.rs). Use these for full-SNARK or Spartan IOP questions, not for standalone-WHIR Solidity gas work. |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ethereum/sol-whir-p3](https://github.com/ethereum/sol-whir-p3) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
