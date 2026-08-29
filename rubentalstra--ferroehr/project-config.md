---
trigger: always_on
description: A pure-Rust, **openEHR-spec-conformant** CDR (ITS-REST 1.1.0 + AQL 1.1) in a single root Cargo workspace, with greenfield PG18-native internals. The openEHR spec + serialization + REST-contract layer is **generated** from the official machine-readable specs; the application is **modern idiomatic Rust of our own design on top of the generated crates**: our own storage, versioning, and AQL engine, validated by the openEHR CNF conformance suite — EHRbase is prior art, not an oracle. **The open-item
---

# CLAUDE.md

A pure-Rust, **openEHR-spec-conformant** CDR (ITS-REST 1.1.0 + AQL 1.1) in a single root Cargo workspace, with greenfield PG18-native internals. The openEHR spec + serialization + REST-contract layer is **generated** from the official machine-readable specs; the application is **modern idiomatic Rust of our own design on top of the generated crates**: our own storage, versioning, and AQL engine, validated by the openEHR CNF conformance suite — EHRbase is prior art, not an oracle. **The open-items tracker is GitHub Issues** (see the Issue workflow below); the public direction + live-status surface over it is the FerroEHR Roadmap board (a GitHub Project view — its readme carries the durable direction themes; see the board paragraph below); deep working plans live in `docs/plans/`. The tracker (issues, PRs, `CHANGELOG.md`, git history) is the authoritative record: closing PR descriptions + issue handoff comments carry the build narrative, the board + milestones carry the roadmap. **There is no ADR/design-doc layer (owner rulings 2026-07-16 + 2026-07-17): decisions live in THIS file, `docs/architecture.md`, and the code; a plan/design markdown is DELETED in the PR that implements it; the vendored specs are the only doc oracle.**

## Repo map

Single workspace. **Crate naming:** `openehr-*` = the openEHR **specification** (generated from the vendored BMM/XSD/OAS — treat as `// @generated`); `ferroehr-*` = the **application** (idiomatic Rust of our own design consuming the `openehr-*` crates). **The `openehr-*` crates are PUBLISHED on crates.io** (their own independent SemVer line — permanently decoupled from the vendored spec versions, which live only in each crate's per-generation pins — the emitted `Generation` enum + generation-module `SPEC_VERSION`s for the generated crates, a literal `SPEC_VERSION` for the hand-written ones; lockstep across all eight): a PR that changes packaged crate content bumps the version in the same PR — `.claude/rules/crates-publishing.md`, machine-enforced by the `crate-version-guard` CI job and the `crate_version_bump_guard.sh` push hook. EHRbase prior art is consulted via its upstream repos / git history, never from an in-tree copy.

- `crates/openehr-base`, `openehr-rm`, `openehr-am`, `openehr-term`, `openehr-lang` — **generated** spec crates (`openehr-codegen -- emit`). `openehr-its` — canonical JSON + **generated** XML (`emit-xml`) + **generated** ITS-REST contract (`emit-rest`) + hand-written runtimes + the hand-written `flat` module (Simplified Formats: FLAT/STRUCTURED/Web Template — the STABLE ITS-REST 1.1.0 Formats sub-spec). `openehr-query` — hand-written AQL lexer/parser/AST. `openehr-adl` — hand-written ADL 2.4 engine (ADL2/cADL/ODIN parser, AOM2 validation, flattener, OPT2, ADL 1.4→2 conversion) over `openehr-am::v2_4`. `openehr-codegen` (BMM/XSD/OAS→Rust generator) is the hand-written tooling.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rubentalstra/FerroEHR](https://github.com/rubentalstra/FerroEHR) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-29 -->
