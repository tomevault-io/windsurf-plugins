---
trigger: always_on
description: This project uses **bd (beads)** for issue tracking.
---

# AI/Agent Instructions for `reflaxe.rust`

## Issue Tracking

This project uses **bd (beads)** for issue tracking.

- `bd prime` — workflow context
- `bd ready` — unblocked work
- `bd show <id>` — details
- `bd update <id> --claim` — claim
- `bd close <id>` — complete
- `bd export -o .beads/issues.jsonl` — write the git-tracked issue export from the embedded DB
- `bd dolt push` / `bd dolt pull` — sync the embedded Dolt DB once a remote is configured

Gotcha: `bd` DB state and `.beads/issues.jsonl` can drift because the JSONL file is an explicit export of the embedded Dolt database.
Before committing bead status changes, run `bd export -o .beads/issues.jsonl` and ensure `.beads/issues.jsonl` is included in the commit when modified.
- Modern Beads migration gotcha: this repo has been migrated to the embedded Dolt backend (`bd context` should report `Backend: dolt`, `mode: embedded`).
  Do not use legacy direct SQLite-style `bd --db .beads/beads.db ...` commands; they can open an empty legacy database and remove/hide the JSONL export.
  If recovery is needed, first copy `.beads/issues.jsonl` to a temp path, then run `bd init --from-jsonl --reinit-local --prefix haxe.rust --skip-agents --skip-hooks --non-interactive`
  and verify `bd status` matches the JSONL counts before mutating issues.
  The modern backend normalizes the configured prefix to `haxe_rust`; when adding children to the historical `haxe.rust-*` roadmap, use explicit IDs with `bd create --force --id ...`.

Milestone plan lives in Beads under epic `haxe.rust-oo3` (see `bd graph haxe.rust-oo3 --compact`).

## Thinking Levels (Bead Labels)

Use a `thinking:*` label on active beads so execution effort matches task risk.

- `thinking:low`
  - Mechanical edits, simple docs cleanup, straightforward renames, obvious wiring.
- `thinking:medium`
  - CI/job plumbing, runner scripts, artifact flow, bounded retry/timeout logic.
- `thinking:high`
  - Parity contracts, gate semantics, dependency graph changes, perf-policy changes, compiler/runtime architecture decisions.
- `thinking:xhigh`
  - Scope-definition changes, release enforcement, provenance-sensitive implementation strategy, or any task where a wrong decision would create misleading 1.0 evidence.

Agent policy:

- When a bead has a `thinking:*` label, match reasoning depth to that label automatically.
- If a claimed bead has no `thinking:*` label, infer one immediately and add it before substantial work.
- `thinking:xhigh` should get a second-pass review before closure.
  - Preferred: an Oracle checkpoint/review.
    - Default Oracle workflow: prepare a detailed prompt for GPT-5.5 Pro in the web UI, including the review questions, relevant file paths, Beads IDs/commands, and any repo bundles to upload (for example a repomix archive). Give that prompt to the user to paste, wait for the user to paste the reply back, then incorporate the findings.
    - Do not use a subagent for Oracle-style review unless the user explicitly asks for one.
  - Acceptable fallback: an explicit written second-pass design review recorded in the bead comments.
- Oracle is a review/escalation tool for `thinking:xhigh`; it is not a substitute for implementation, tests, or CI evidence.

## Product Source of Truth

- Requirements + architecture: `prd.md`
- Target: **Haxe 4.3.7 → Rust** via Reflaxe

## Strategic Goal

- Primary long-term goal: make `reflaxe.rust` the best way to write production Rust outside of writing raw Rust directly, by combining Haxe ergonomics with Rust-level performance, safety, and readability, while preserving an explicit portable path through Haxe when users want cross-target portability.

## Typeful Haxe and Rust Output Quality

- Treat well-typed Haxe as the source language contract. Compiler, runtime, std overrides, tests, and examples should use concrete types, `typedef` schemas, abstracts/newtypes, enum abstracts, typed enums, and GADT-style typed enum patterns where Haxe can express them. Avoid strings as domain models when a stronger representation is practical.
- Keep stringly typed values at real boundaries only: JSON/protocol IO, CLI/env/filesystem inputs, metadata names, target syntax tokens, or upstream API compatibility points. Convert those values into typed structures immediately after the boundary and keep downstream code typed.
- Use macros when they improve correctness or maintainability: deriving repetitive validators/bridges, centralizing typed target metadata, enforcing profile/runtime contracts, or preventing schema drift. Avoid clever macro machinery whose generated shape is hard to inspect, hard to test, or harder for haxe.rust to lower cleanly.
- Adapt Rust/Codex-style architecture into Haxe idioms rather than mechanically mirroring Rust. Prefer Haxe abstractions that make invalid states unrepresentable while still giving the backend enough typed information to emit native Rust.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fullofcaffeine/reflaxe.rust](https://github.com/fullofcaffeine/reflaxe.rust) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
