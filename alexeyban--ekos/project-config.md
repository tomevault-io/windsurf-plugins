---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Status

EKOS has an implemented Rust (2024 edition) Cargo workspace — this is not a design-phase repo.
Read `devlogs/devlog_*.md` (numbered chronologically — the highest-numbered file is the latest,
`devlog_148.md` as of this writing) before starting non-trivial work: they are the project's
long-term memory and record what shipped, why, and what was learned. `TODO.md` tracks the
phase-by-phase roadmap; RFCs are split across two locations for historical reasons, not a
meaningful distinction — `docs/rfcs/` (repo root) has `0001`–`0024`,
`ekos/docs/rfcs/` has `0025`+. **Check both directories for the highest existing number before
picking one for a new RFC** — two RFCs have already collided on the same number once (0027) from
sessions that only checked one location.

**The Cargo workspace root is `ekos/`, not the repo root** — there is no top-level `Cargo.toml`.
`benchmark/` and `tests/integration/` are separate Cargo workspaces that depend on the `ekos/`
crates by path.

## Commands

```bash
# Main workspace (run from ekos/, or pass -p <crate>/--manifest-path from elsewhere)
cd ekos
cargo build --workspace
cargo test --workspace
cargo test -p ekos-ledger                    # one crate, e.g. ekos-ledger, ekos-plugin-git
cargo test -p ekos-ledger some_test_name     # one test
cargo clippy --workspace -- -D warnings      # CI fails on any warning
cargo fmt --check                            # CI checks formatting, doesn't fix it

# Integration tests (separate workspace, depends on ekos/ crates by path)
cd tests/integration && cargo test

# Benchmarks (separate workspace, Criterion)
cd benchmark && cargo bench
cargo bench --bench ledger_write             # one benchmark file

# CLI (binary crate name is `ekos`, package is crates/cli)
cargo run -p ekos -- init
cargo run -p ekos -- build && cargo run -p ekos -- recover && cargo run -p ekos -- resolve \
  && cargo run -p ekos -- compile && cargo run -p ekos -- commit   # full pipeline, in order
cargo run -p ekos -- doctor
cargo run -p ekos -- ask "<question>"        # compiled REASON answer (--classic / --explain)
cargo run -p ekos -- query find "<text>" --mode hybrid   # SEARCH: BM25 + vector, RRF-fused (RFC 0118/0125)
cargo run -p ekos -- ekl "FIND Table WHERE ..."          # Enterprise Knowledge Language
cargo run -p ekos -- identity scan          # cross-system candidate matches (RFC 0029)
cargo run -p ekos -- marketing publish      # devlog -> tweet -> approval -> X (RFC 0030)
cargo run -p ekos -- mcp serve --workspace <dir>
cargo run -p ekos -- docs generate --layout curated --output doc   # README/Architecture/API/
                                             # SequenceDiagrams + per-entity pages (RFC 0035/0037/0042)
cargo run -p ekos -- simulate <scenario.yaml>       # World Engine: load + run a scenario (RFC 0047-0055)
cargo run -p ekos -- replay <scenario.yaml>         # read back a previously recorded simulation, read-only
```

CI (`.github/workflows/ci.yml`) runs build+test+clippy+fmt from `ekos/` and `cargo bench` from
`benchmark/` on every push/PR to `main`. Match these locally before pushing.

## High-Level Architecture

EKOS is a **compiler for enterprise knowledge**, not a database or document store. It observes
enterprise systems without interpreting them, compiles those observations through deterministic
passes into a Canonical Knowledge Model, and stores the result in an append-only ledger where
every conclusion carries the evidence it was derived from.

```
Enterprise Systems → Observation Layer → Knowledge Compiler → Canonical Knowledge Model (CKM)
                                                                          ↓
                              AI/Apps ← Knowledge Runtime (read-only) ← Semantic Knowledge Ledger
```

This maps directly onto the `ekos init/build/recover/resolve/compile/commit` CLI pipeline: each
verb is a compiler stage, run in that order, writing artifacts the next stage consumes.

### Crate map (`ekos/crates/`)

| Crate | Role |
|---|---|
| `compiler-core` | `Compiler`, `PassManager`, `Scheduler`, `Diagnostics`, `EkosConfig` — the pipeline that drives every pass over a `PassContext` |
| `compiler-sdk` | Public traits for extending the compiler |
| `observation-sdk` | `Observer` trait — the contract every connector implements, returning an `ObservationPackage` of content-addressable `ObservationArtifact`s |
| `artifact` | Artifact types + `ArtifactStore` (loose JSON, or packed segments post RFC 0015) |
| `kir` | Knowledge Intermediate Representation — the typed output of knowledge-recovery passes, input to the semantic compiler |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [alexeyban/EKOS](https://github.com/alexeyban/EKOS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-19 -->
