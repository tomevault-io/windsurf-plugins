---
trigger: always_on
description: This file provides guidance to Codex (Codex.ai/code) when working with code in
---

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in
this repository.

## Project

`opi` is a Rust AI Agent toolkit with a terminal-first coding Agent as its
Reference Product. It reimplements selected ideas from
[earendil-works/pi](https://github.com/earendil-works/pi), but it is not a
line-by-line port or a pi compatibility layer.

## Sources of truth

Use the narrowest authoritative source for each claim:

- `docs/opi-spec.md` is the normative source for durable product direction,
  architecture invariants, admission gates, and strategic priority.
- `docs/CONTEXT.md` owns the domain language for architecture, extension
  runtime, command execution, evidence, and authority boundaries.
- `README.md`, generated `opi --help`, crate documentation, manifests, schemas,
  fixtures, and source own current product and protocol facts.
- `Cargo.toml` and crate manifests own workspace topology, versions, Rust
  edition, MSRV, and dependency declarations.
- `CHANGELOG.md` owns release history and unreleased user-visible changes.
- `docs/realign/` and `.repo/pi-0.84.1` are non-normative inward evidence;
  `docs/research/` is non-normative outward evidence.
- `.opi-impl-state.json` and `docs/snapshots/` own implementation progress and
  completed delivery history. Do not record progress in `docs/opi-spec.md`.

When documentation has an English/Chinese counterpart, update both in the same
change or state why synchronization is unnecessary. `CLAUDE.md` is a symlink to
`AGENTS.md`, so the repository keeps exactly one guidance file; edit
`AGENTS.md` and never `CLAUDE.md` directly. On a filesystem without symlink
support a fresh clone materializes `CLAUDE.md` as a text file containing
`AGENTS.md`; `scripts/opi-doc-check.py` warns on that state and fails on real
content drift.

## Design boundaries

- Keep the Agent Core small and deep. Mechanism belongs below policy; terminal,
  workflow, benchmark, and user-policy opinions do not belong in core crates.
- Dependencies point inward toward the smallest stable interface. A new public
  seam needs intrinsic state-machine value or at least two real adapters or
  consumers with shared conformance tests.
- Optional Opi workflows belong in the Extension Ecosystem. Agent-neutral
  capabilities should begin as Independent Companions with Agent-neutral
  contracts.
- Prefer Rust-native correctness: enums for closed states, explicit ownership,
  typed errors, bounded concurrency, and fail-closed validation at authority,
  protocol, adapter, and permission boundaries.
- Do not add a feature flag, trait, crate, config key, compatibility layer, or
  abstraction for hypothetical future use.

Consult the spec before answering scope or architecture questions. If a request
would contradict a normative clause, stop and ask the user whether they intend
to revise the specification.

## Workspace layout

All crates use lockstep workspace versioning and Rust edition 2024.

```text
opi-ai       (no internal deps)       - provider-neutral LLM API
opi-tui      (no internal deps)       - terminal UI components
opi-eval     (no internal deps)       - unpublished Independent Companion eval toolkit
opi-agent    -> opi-ai                - product-neutral Agent runtime
opi-protocol (no internal deps)       - command-execution protocol
opi-sandbox  -> opi-protocol          - standalone restriction SDK/CLI
opi-coding-agent -> opi-ai, opi-agent, opi-protocol, opi-tui - opi binary and coding harness
```

Internal dependencies must be declared in root `[workspace.dependencies]` and
referenced with `{ workspace = true }`. Publishable path dependencies also need
a version. Do not duplicate workspace-owned package metadata in crate manifests.

## Dependency and supply-chain safety

- Treat changes to `Cargo.toml`, `Cargo.lock`, Cargo features, `build.rs`, native
  dependencies, and proc-macro dependencies as reviewed code.
- Verify third-party APIs against the exact locked crate version or its
  authoritative documentation; do not guess from another version.
- Never hand-edit `Cargo.lock`. Regenerate it with Cargo and review unexpected
  direct and transitive changes.
- Do not introduce or broaden default features, build scripts, native code, or
  proc macros without explaining the portability, authority, and supply-chain
  impact.

## Project workflow

The canonical workflow and skill-selection policy live in
`.claude/skills/README.md` and `.claude/skills/README.zh.md`. All `opi-*` skills
require explicit user invocation; use `opi-workflow` only when the user asks for
workflow routing.

- `opi-realign` gathers pinned pi alignment evidence.
- `opi-research` gathers outward capability evidence.
- Human-led shaping updates the normative spec or a registered supplemental
  source.
- `opi-implement plan` is the admission gate for implementation work;
  `opi-implement` alone owns `.opi-impl-state.json`.
- `opi-audit`, `opi-eval`, and `opi-remediate` provide assurance and correction.
- `opi-document`, `opi-release`, and `opi-slim-tests` own their named workflows.

Do not hand-edit `.opi-impl-state.json`, create a competing implementation
ledger, or treat arbitrary `docs/superpowers/specs/` files as normative.

## Working principles


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [OdradekAI/opi](https://github.com/OdradekAI/opi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-29 -->
