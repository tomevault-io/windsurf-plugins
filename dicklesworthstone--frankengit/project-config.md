---
trigger: always_on
description: This file is normative for humans and software agents working in this repository. It applies even when a task appears small. FrankenGit is pre-implementation; the principal risk is creating a convenient early abstraction that contradicts the final system and becomes expensive technical debt.
---

# AGENTS.md — FrankenGit Contributor and Coding-Agent Contract

This file is normative for humans and software agents working in this repository. It applies even when a task appears small. FrankenGit is pre-implementation; the principal risk is creating a convenient early abstraction that contradicts the final system and becomes expensive technical debt.

## 1. Mission

Build a clean-room, pure-Rust, Git-compatible forge whose canonical state, transfer, workspaces, graph intelligence, recovery, verification, and agent authority remain coherent from one embedded node to a hosted multi-region service.

The project values:

- exact behavior over vague compatibility;
- final abstractions over throwaway scaffolds;
- algorithmic performance over unsafe shortcuts;
- immutable evidence over confident prose;
- typed refusal over panic, silent fallback, or partial publication;
- local reproducibility over hosted-service dependence;
- negative evidence over repeating failed ideas.

## 2. Constitutional hierarchy

Before a material change, read the relevant portions of:

1. [`docs/NORMATIVE_PROTOCOL_CONTRACTS.md`](docs/NORMATIVE_PROTOCOL_CONTRACTS.md)
2. [`docs/DEPENDENCY_AND_MEMORY_SAFETY_CONSTITUTION.md`](docs/DEPENDENCY_AND_MEMORY_SAFETY_CONSTITUTION.md)
   - binding sibling-integration contract: [`docs/ASUPERSYNC_AND_FRANKENSQLITE_INTEGRATION_PROFILE.md`](docs/ASUPERSYNC_AND_FRANKENSQLITE_INTEGRATION_PROFILE.md)
3. [`COMPREHENSIVE_PLAN_FOR_THE_DESIGN_OF_FRANKENGIT.md`](COMPREHENSIVE_PLAN_FOR_THE_DESIGN_OF_FRANKENGIT.md)
4. the focused subsystem specification under `docs/`
5. `VERIFY_SPEC.md` and `SECURITY_THREAT_MODEL.md`
6. machine-validated `registries/`

If these disagree, stop and surface the contradiction. Do not implement the most convenient interpretation.

## 3. Non-negotiable construction rules

### 3.1 Pure Rust and memory safety

- Every first-party crate must use `#![forbid(unsafe_code)]`.
- Do not add an unsafe boundary crate, local lint exception, raw-pointer shortcut, inline assembly, or FFI shim.
- Do not link C/C++ libraries or system native libraries to obtain Git, compression, TLS, crypto, database, search, graph, or sandbox behavior.
- Do not invoke `git`, `libgit2`, JGit, Dulwich, another VCS engine, or a helper that hides one in production.
- Upstream Git may run only in pinned, sandboxed, explicitly non-production differential/conformance lanes.
- Unsupported behavior returns a typed refusal. It never falls back secretly.

### 3.2 One runtime

- Asupersync is the sole async runtime.
- Do not add Tokio, async-std, smol, executor-lite, or an ecosystem dependency that brings an alternate runtime into production.
- Long-lived work owns children through regions and closes to quiescence.
- Cancellation is request → drain → finalize; dropping a future is not a complete protocol.
- Effects that acquire responsibility use typed obligations: reserve/commit/abort as the two-phase boundary, plus explicit acknowledgement for externally observed effects.

### 3.3 Closed dependency universe

- Prefer std, Asupersync, and stable factored FrankenSuite crates.
- External crates must be fundamental, pure Rust, narrowly scoped, registry-approved, and justified by marginal capability.
- Do not add a dependency because it makes a prototype shorter.
- Record transitive unsafe, build scripts, proc macros, native tools, license, version policy, alternatives, audit surface, and removal path.
- One `Cargo.lock` and one compatible FrankenSuite/runtime constellation are required.
- Never commit an unpublished local path dependency for a release-facing crate.

### 3.4 Latest nightly, reproducibly

- Use the dated nightly in `rust-toolchain.toml`, not a floating `nightly` string.
- A toolchain advancement is a material change: run compatibility, conformance, determinism, and performance checks; record regressions/negative evidence.

## 4. Final-abstraction slice doctrine

A new crate/module appears only with one real vertical slice of its final abstraction.

Forbidden substitutes include:

- an in-memory `HashMap` described as durable storage;
- an empty crate with future TODOs;
- a fake parser that accepts only fixtures but is wired as the production API;
- a mutable local repository treated as canonical truth;
- a second database whose rows compete with the authority-head decision stream;
- a workflow file containing logic unavailable through repository-owned commands;
- a model/graph score used as authorization;
- a decoder result accepted without original commitments;
- a benchmark-only optimization without output/ordering equivalence.

A subset is acceptable when its unsupported surface is typed and the implemented path already has the final ownership, failure, cancellation, and evidence boundaries.

## 5. Canonical-state rules

### 5.1 Authority

- Only successful conditional replacement of the exact predecessor `RepositoryAuthorityHead` publishes repository state.
- Routing, gossip, local SQLite rows, materializations, indexes, and caches are hints/projections.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Dicklesworthstone/frankengit](https://github.com/Dicklesworthstone/frankengit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-21 -->
