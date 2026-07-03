---
trigger: always_on
description: <!-- SPDX-FileCopyrightText: 2026 Blackcat Informatics® Inc. <paudley@blackcatinformatics.ca> -->
---

<!-- SPDX-FileCopyrightText: 2026 Blackcat Informatics® Inc. <paudley@blackcatinformatics.ca> -->
<!-- SPDX-License-Identifier: CC-BY-4.0 -->

# AI Developer Agent Guide (AGENTS.md)

Welcome, AI Agent! This file is your behavioral contract and instruction manual for
contributing to the PurRDF repository.

---

## 1. What this repository is

PurRDF is the **RDF 1.2 toolkit** that several downstream projects (notably
[`gmeow-ontology`](https://github.com/Blackcat-Informatics/gmeow-ontology)) use as
their **data-carrier backbone**. It must stay fast, deterministic, and boring:
one engine, one behavior, carried verbatim into Rust, Python, WebAssembly, and C.

Crate map (all under `crates/`, published names in `Cargo.toml`):

| Crate | Role |
|---|---|
| `purrdf` | Umbrella facade (RDF surface at root; `slice`/`shapes` as modules) |
| `purrdf-rdf` (`crates/rdf`) | Native text/XML/JSON-LD codecs, GTS adapters, describe, canonicalization |
| `purrdf-core` (`crates/rdf-core`) | Interned IR kernel, diagnostics, store traits, provenance, RDFC-1.0 |
| `purrdf-gts` (`crates/gts`) | GTS container engine (CBOR log, BLAKE3, COSE) |
| `purrdf-sparql-{algebra,eval,results}` | SPARQL 1.1/1.2 parser, evaluator, results |
| `purrdf-shapes` (`crates/shapes`) | SHACL validation (full Core + SHACL-SPARQL) |
| `purrdf-shex` (`crates/shex`) | ShEx 2.1 schemas + validation |
| `purrdf-slice` (`crates/slice`) | Slice catalog, artifacts, ownership analysis |
| `purrdf-iri`, `purrdf-xsd`, `purrdf-events` | Zero-dependency foundations |
| `purrdf-wasm`, `purrdf-capi`, `bindings/python` | WASM, C-ABI, and PyO3 bindings |

## 2. Hard constraints (violating these fails CI or review)

* **NO Cargo features, ever.** The workspace has zero feature flags and
  `scripts/check-no-features.py` gates CI. PurRDF is a carrier; optionality
  changes semantics per consumer, which is forbidden. Do not add `[features]`,
  optional deps, or `cfg`-gated behavior differences.
* **Kernel ring-fence.** `purrdf-core` must never depend on oxigraph or PyO3.
  `purrdf-iri`, `purrdf-xsd`, and `purrdf-events` must keep **zero runtime
  dependencies**.
* **Everything is wasm-able.** Every release crate (all 13 published crates,
  `purrdf-wasm` included) must build for `wasm32-unknown-unknown` — CI
  hard-fails otherwise (`make wasm` locally). Never add a dependency that
  drags in threads, the filesystem, C toolchains, or wall-clock/RNG syscalls
  on the wasm path; crypto stays pure-Rust for exactly this reason.
* **Byte determinism.** Serializers and the GTS writer are byte-deterministic.
  If your change alters emitted bytes, you must update the affected goldens and
  say why in the PR. Never introduce iteration-order, time, or RNG dependence
  into output paths (hashers are fixed-key `ahash` for this reason).
* **Conformance corpora are the contract**: W3C SPARQL 1.1
  (`crates/sparql-conformance`), the W3C SHACL suite (`vectors/shacl/`), the
  shexTest v2.1.0 suite (`vectors/shexTest/`), the first-party SHACL corpus
  (`crates/shapes/corpus/`), RDFC-1.0 fixtures
  (`crates/rdf/tests/fixtures/rdfc/`), and the **frozen** GTS vectors in
  `vectors/` (shared byte-exact with the other GTS engines — never regenerate or
  "fix" them here; the GTS wire format is governed in `gmeow-gts`). Harnesses
  assert exact counts and enforce XPASS discipline on their xfail ledgers —
  see [`docs/CONFORMANCE.md`](./docs/CONFORMANCE.md) for the scoreboard.
* **PurRDF is NOT an ontology — it mints no vocabulary IRIs, ever.** Every
  vocabulary the library reads or writes (slice manifests, statement-metadata
  downcast, box roles, language retagging, SPARQL extension-function
  namespaces, standpoint predicates, json_schema namespaces) is
  **caller-supplied configuration with no fabricated default**: a feature
  exercised without its vocabulary hard-errors or stays inactive. Never
  hardcode a `blackcatinformatics.ca` namespace in library code (the GMEOW
  ontology is a *consumer*; the dependency arrow never points from purrdf to
  it). Test fixtures use `example.org`.
* **Generated artifacts** under `generated/` are projections — never hand-edit;
  regenerate via `make metadata` (`scripts/check-generated.sh` gates drift).
* **Dependency versions live in one place**: `[workspace.dependencies]` in the
  root `Cargo.toml`. Member crates use `dep.workspace = true`. Do not pin a
  version inside a member manifest.
* **Lints are workspace-inherited** (`[workspace.lints]`, clippy pedantic +
  nursery). `cargo clippy --workspace --all-targets` must be warning-free.
  Prefer fixing code over `#[allow]`; a genuinely-right allow must be tightly
  scoped and carry a reason comment.
* **SPDX headers** on every source file: `MIT OR Apache-2.0` (docs may be
  `CC-BY-4.0`).

## 3. Commands

```bash
make check      # the full local gate: fmt, clippy, build, tests, hygiene
make test       # cargo test --workspace
make metadata   # regenerate + verify generated artifacts
make bench      # criterion benchmarks (report-only; not a gate)
```

Toolchain: `rust-toolchain.toml` pins **stable** and the workspace is
nightly-free by policy (`rust-version` in `Cargo.toml` is the enforced MSRV

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Blackcat-Informatics/purrdf](https://github.com/Blackcat-Informatics/purrdf) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-02 -->
