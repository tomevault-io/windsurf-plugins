---
trigger: always_on
description: A minimal (zero-knowledge Virtual Machine, which is actually not ZK in the real sense, i.e. it's only a snark, not a zk-snark).
---

# AGENT.md

## What this is

A minimal (zero-knowledge Virtual Machine, which is actually not ZK in the real sense, i.e. it's only a snark, not a zk-snark).

- `doc/leanvm/` is the LaTeX project describing the machine ISA and the snark that proves it. Its root is `doc/leanvm/main.tex`; build it with `cd doc/leanvm && latexmk -pdf main.tex`, which writes to the gitignored `doc/leanvm/.build/`. Sections live in `doc/leanvm/body/`, numbered `01`..`10` plus the lettered annexes `a` (ring switching), `b` (the PCS), `c` (Flock), and `d` (novel basis and additive NTT), and every symbol is defined once in `doc/leanvm/preamble/macros.tex`. If latexmk fails oddly (a bibtex error, or a missing `main.log`) right after inputs are renamed or `refs.bib` is edited, remove `doc/leanvm/.build` and rerun; it has not reproduced on unchanged inputs. **Drafting one section:** each section file carries a `% !TeX root` comment pointing at its generated driver in `doc/leanvm/drafts/`, so the LaTeX build key (`F5`, or the extension's `cmd+alt+b`) compiles only that section, numbered as in the full document and with cross-references and citations resolved against `.build/main.aux`; in `main.tex` the same key builds everything. Run `doc/leanvm/make-drafts.sh` after adding, renaming or renumbering a section.
- `doc/xmss/` is the standalone specification of the concrete XMSS instance implemented by `crates/xmss`.
- `doc/sphincs/` is the standalone specification of the concrete SPHINCS+ instance we would use instead of XMSS where statelessness matters; its root is `doc/sphincs/main.tex`, built the same way as `doc/xmss`, and implemented by `crates/sphincs`. It shares XMSS's hash function, tweakable hash and target-sum code, so an aggregator implements one primitive.
- `formal/xmss/` is a Lean 4 proof (over VCVio) of that instance's classical random-oracle security, `xmss_has_127_bits_of_classical_security`. `XmssSecurity/Statement.lean` is the only module a reviewer has to read: the concrete parameters, the byte layout of every hash input, the three algorithms, the game, and the claim. `lake exe cache get` once, then `lake build`. SPHINCS has no formalization; its security section is a target, not a theorem.
- The one hash function is BLAKE2s, in `primitives::hash`: scalar, streaming, keyed, and a lane-transposed batched form for the PCS Merkle tree. The VM proves one compression per opcode, and BLAKE2s takes the byte counter and final-block flag as ordinary compression inputs, so a single opcode is a complete hash for any length, with no tree structure to reproduce in-circuit.
- `crates/lean_compiler/zkDSL.md` documents the (pythonic) zkDSL (that compiles to the ISA that our VM runs, and that our snark proves).

Primary goal:
- Aggregate XMSS (stateful hash based signatures), via a snark proving knowledge soundness of the signatures, against a common message and a lust of public keys
- Further aggregate n previously aggregated signatures, which is performed by a recursive snark, that proves "I know n sub-proofs that are valid and the union of the public keys they handle contains the list of public keys I am given in public input". 

## Layout

Dependency order, leaves first:

| crate             | role                                                                   |
| ----------------- | ---------------------------------------------------------------------- |
| `parallel`        | thread pool (below)                                     |
| `zk_alloc`        | proving arena (below)                                    |
| `primitives`      | field kernels (NEON/AVX), bit transposes, multilinear helpers, streaming stores, `bench` |
| `fiat_shamir`     | VM-native `FiatShamirState` + prover/verifier transcript                |
| `pcs`             | additive NTT, Merkle, ring switch, stacked WHIR                    |
| `flock`           | batched R1CS over GF(2) for BLAKE2s: zerocheck + lincheck               |
| `lean_vm`         | arithmetization: tables, bus, constraints, `cpu::prove`/`verify`       |
| `lean_compiler`   | zkDSL (Python subset) → ISA                                            |
| `xmss`            | XMSS over BLAKE2s; an independent leaf, consumed only by `rec_aggregation` |
| `sphincs`         | the stateless SPHINCS+ instance of `doc/sphincs`; an independent leaf, consumed only by `rec_aggregation` |
| `rec_aggregation` | recursive XMSS and SPHINCS aggregation: the one guest, the entry points `src/lib.rs` re-exports, the benchmarks |

`src/lib.rs` is the public API and the only thing a user imports: every crate above is `publish = false`, so a new user-facing item is a re-export there. `src/main.rs` is the benchmark CLI, `tests/api.rs` the end-to-end use of the API; guests are zkDSL under `crates/rec_aggregation/guests/`.

## Building / Testing / Formatting

- `.cargo/config.toml` pins `-C target-cpu=native` and `-D warnings` for rustdoc
- always run in `--release` mode any test or benchmark touching the VM (the zkDSL compiler stack-overflows in `debug` mode)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [leanEthereum/leanVM](https://github.com/leanEthereum/leanVM) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
