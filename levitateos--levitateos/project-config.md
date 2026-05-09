---
trigger: always_on
description: This file is intentionally compact. Priority is preventing policy violations and reward-hacking behavior.
---

# Repository Guidelines (Policy-First)

This file is intentionally compact. Priority is preventing policy violations and reward-hacking behavior.

## 0) Prime Directive
- Do not optimize for green checks. Optimize for true checkpoint correctness and reproducibility.
- Never hide failures. Fix root causes.
- If a command passes only because of suppression/masking/fallback tricks, treat it as a failure.

## 1) Workspace and Ownership
- New work belongs in: `distro-variants/*`, `distro-builder`, `distro-contract`, `distro-spec`, `testing/*`, `xtask`.
- Legacy crates are read-only unless explicitly requested for scoped compatibility:
  - `leviso/`, `AcornOS/`, `IuppiterOS/`, `RalphOS/`.

## 1.1) Recipe vs Orchestrator Boundary (Strict)
- `recipe` is the canonical owner for package/source knowledge.
- Rust/TOML orchestration is the canonical owner for sequencing, policy, contract checks, and artifact topology.
- Keep the boundary hard:
  - Recipes own versions, URLs, checksums, torrent links, package lists, trust-marker semantics, default paths, and acquire/build/install/remove logic.
  - Rust owns recipe selection, build/output roots, product/release/scenario sequencing, checkpoint policy enforcement, and contract validation.
- Rust must stay dumb about recipe contents.
- Forbidden in Rust/TOML default paths:
  - duplicating release URLs, checksums, torrent URLs, or package lists solely to inject them into recipes
  - constructing large required `--define` maps for canonical recipe execution
  - hardcoding recipe-specific trust-marker names, remote layout rules, or package expectations that belong in the recipe
  - treating a user-facing recipe as an internal worker script while still exposing it like a normal recipe
- Required model for user-facing recipes:
  - the canonical recipe must run with zero required defines for the normal case
  - overrides may exist, but they must be optional and additive
  - a user must not have to look up upstream metadata manually just to run the recipe
- If a script truly requires injected defines, then it is an internal worker, not a normal recipe.
- Internal workers must be clearly marked as such and must not be the default public UX when a canonical recipe should exist.
- When Rust and a recipe both know the same source/package fact, that is an ownership bug.
- Fix ownership bugs by moving factual knowledge into the recipe and reducing Rust to orchestration only.
- Desired mental model:
  - recipe = smart, self-contained source/package knowledge
  - Rust = dumb orchestrator that runs the right recipe at the right time

## 2) Hard Ban: Legacy Binding
- Never wire checkpoint/rootfs/tooling paths to legacy crate downloads outputs.
- Forbidden examples (non-exhaustive):
  - `*/downloads/rootfs` from legacy crates
  - `leviso/downloads/.tools` (or any legacy crate equivalent)
  - dynamic fallback/autodiscovery to `*/downloads/.tools`
- Required guard before build/commit:
  - `cargo xtask policy audit-legacy-bindings`
  - alias: `just policy-legacy`
- Any violation is a hard failure.

## 2.1) Guard Placement (Authoritative Boundary)
- Guard placement is valid only at executable entrypoints that perform work (`distro-builder`, `testing/install-tests`, `xtask` commands that build/test/scenario-run).
- `just` checks are convenience only and must never be the sole enforcement layer.
- A command path that can build/test artifacts without running policy guards is a policy bug and must be fixed immediately.
- Required model:
  - preflight guard runs inside the executable command path, before any artifact mutation or QEMU/test launch
  - fail-fast on guard violation with cheat-guard diagnostics
  - no "best effort continue" after guard failure
- Treat wrapper-only guard wiring as insufficient even if current developers usually use wrappers.

## 3) Checkpoint Vocabulary (Canonical)
- Use canonical checkpoint names only where the conformance ladder is the real owner:
  - `00Build`, `01Boot`, `02LiveTools`, `03Install`, `04LoginGate`, `05Harness`, `06Runtime`, `07Update`, `08Package`
- Rings, products, releases, and scenarios own manifests, filetree layout, and orchestration.
- Do not introduce new legacy alias families, `sNN_*`, or checkpoint-numbered owners outside explicit compatibility or reporting surfaces.

## 4) Checkpoint Artifact Split (Current Output Family)
- Every checkpoint writes non-kernel outputs only into its own directory family:
  - `.artifacts/out/<distro>/sNN-<checkpoint-name>/...`
- Cross-checkpoint reuse of non-kernel artifacts is forbidden in release/hardening mode.
- Kernel artifacts are the only shared artifacts across checkpoints in release/hardening mode.
- While `9.2` bypass mode is active, `02LiveTools` may intentionally carry forward-looking non-kernel payload for filesystem design convenience.
- No cross-checkpoint symlinks/copies/manual post-build surgery to fake checkpoint outputs.

## 5) Checkpoint Envelope (Mode-Aware)
- A checkpoint artifact must contain exactly what that checkpoint needs in release/hardening mode.
- Missing required payload = fail.
- Carrying later-checkpoint payload = fail in release/hardening mode.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LevitateOS/LevitateOS](https://github.com/LevitateOS/LevitateOS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
