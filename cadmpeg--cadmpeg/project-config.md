---
trigger: always_on
description: - Write dense, direct technical prose. No hype, filler, rhetorical framing, or emphasis cadence. Use ASD-STE100.
---

- Write dense, direct technical prose. No hype, filler, rhetorical framing, or emphasis cadence. Use ASD-STE100.
- Specs state the settled format model as fact. Never qualify it with corpus, sample, experiment, or provenance language.
- Specs contain byte semantics and invariants only. Put genuine unknowns in `docs/formats/*-open-items.md`.
- Open-items fields are Question, Known, Need, Conflict, and Note only. Delete a resolved item in the same change that writes the answer into the specification. Do not keep a Resolved part.
- Do not treat finite evidence as an unknown. Do not put research history, project status, implementation bugs, or export behavior in specs.
- When moving code, update callers to import from the owning module. Do not retain old paths through top-level or orchestration re-exports.
- Commit early, commit often.

Multi-agent repository etiquette:

- One worktree and one branch per agent. Do not edit or build inside another agent's worktree.
- Unstaged changes you did not make belong to another agent. Do not commit them, revert them, or bypass hooks because of them.
- Use `--no-verify` only with the reason stated in the commit body.
- In a conflicted merge, restore a file from a merge stage with `scripts/restore-merge-stage.sh`, not with `git checkout` or `git restore`.

Test placement policy:

- Route a test by ownership of the asserted invariant, not by counting function calls. High-level setup, including `Codec::decode`, does not make a test integration by itself.
- Unit tests live under the production owner: small inline `#[cfg(test)] mod tests { ... }`; `foo.rs` plus `foo/tests.rs` with a final `#[cfg(test)] mod tests;`; or `foo.rs` plus `foo/tests/mod.rs` with semantic children for one large cohesive suite.
- Semantic test submodules such as `foo/tests/parsing.rs` are allowed. Never use numbered names such as `tests_1.rs`.
- Integration tests live under `src/integration_tests.rs` or a semantic `integration_tests/` tree. Use them for codec-wide decode, encode, or round-trip; multi-module composition; the crate facade; or cross-module admission, validation, or fidelity work with no single owner. "Unclear" is not integration by default.
- Test support lives under `src/test_support.rs` or a semantic `test_support/` tree. Single-owner helpers stay with that owner. Shared golden and round-trip helpers use `cadmpeg-test-support`. Do not add a new support crate. Test support is not exempt from size convergence.
- Golden tests live under `src/golden_tests.rs` or a semantic `golden_tests/` tree. They are excluded from test-file size limits. Do not regenerate snapshots during test moves.
- A crate root may expose only three test-only entry points: `golden_tests`, `integration_tests`, and `test_support`. Unit-test modules are declared by their production owners.
- Do not split production only because a cohesive test suite is large. Split production only when an independent review finds a real responsibility seam.
- Test moves preserve behavior. Do not change assertions, expected values, byte literals, tolerances, snapshots, test names, or ignored status.
- Feature work and test movement use separate commits.
- Prohibited final-tree forms: `crates/*/src/tests.rs`; `#[path]` that includes a test-only module; numbered test files; one oversized `integration_tests.rs` or `test_support.rs`; and production re-exports that preserve obsolete paths after a module split.

Build and test operations:

- Run several tests in one invocation with filters after the separator: `cargo test -- name_a name_b`. Plain `cargo test name_a name_b` fails with `unexpected argument`. Fast suite: `cargo test-fast`. Regenerate golden snapshots after an intended change: `UPDATE_GOLDEN=1 cargo test-fast golden`, then review the diff.
- Build and test with `-q`: `cargo build -q`, `cargo test -q` (`cargo test-fast` is quiet already). The flag removes the `Compiling` status lines and condenses the per-test list to one character per test; warnings, errors, and failure detail still print in full. A quiet build with exit status 0 is a completed build — do not rerun it verbosely to confirm. Use `--verbose` only when the compile plan itself is the question.
- The pre-commit gate scopes clippy and tests to the staged crates plus their workspace dependents. Triage a lint finding once and apply a targeted `#[allow]` with a comment; do not rerun the full gate against code you did not touch.
- The `JsonSchema` derives are behind a `schema` feature that is off by default in `cadmpeg-ir`, `cadmpeg-core`, and `cadmpeg-asm`. Codec-private records do not generate schemas; CADIR represents them through `NativeRecord`. `cargo test-fast` does not build them. After changing an IR or native record type run `cargo test -p cadmpeg-ir --features schema --lib`. Do not add `JsonSchema` to a bare `derive` list; use `#[cfg_attr(feature = "schema", derive(JsonSchema))]`.
- Changes to `cadmpeg-ir` or `cadmpeg-core` fan out to every codec crate and diverge all goldens. Add struct fields through `Default` or constructor helpers and plan the fan-out before editing.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cadmpeg/cadmpeg](https://github.com/cadmpeg/cadmpeg) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-25 -->
