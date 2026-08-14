---
trigger: always_on
description: Rust CLI that gates AI-agent shell commands: parse bash into role-tagged
---

# AGENTS

Rust CLI that gates AI-agent shell commands: parse bash into role-tagged
fragments, match each against rules, compose one verdict. Single binary,
edition 2021, toolchain pinned in `rust-toolchain.toml`.

## Two standing goals on every task

The user drives product features and their request is the priority — but carry
two goals into *every* task. When either is the lowest-error path to what the
user asked, fold it into the same task without asking first; surface the rest as
follow-ups (see "Workflow").

1. **Engineer the context for next time.** Make the next agent (and you) see
   more for less: realistic end-to-end tests that exercise what the user
   actually sees — especially when they report a bug existing tests missed (the
   `tests/e2e` suite drives the compiled binary, see "Tests") — scripts and
   skills that automate repetitive steps and shrink their output to signal, and
   terse `AGENTS.md` notes capturing what the code doesn't make obvious.
2. **Engineer the codebase and environment.** Be the engineer the user isn't:
   prioritize the technical initiatives that keep the codebase clean,
   maintainable, and repeatable, and keep environment setup automated and
   consistent (`just bootstrap` from a clean clone). Strict quality gates plus
   local/CI parity (same checks, same pinned toolchain in `rust-toolchain.toml`)
   make results repeatable — not "works on my machine." A clean base and a
   reproducible environment are usually how the user's feature ships with a low
   error rate.

## Stack and composition

- **Product shape:** a command-line tool — a single installable binary users run
  as `allowlister` (and as an agent harness hook). Composed from the create-repo
  skill's `shapes/cli.md`: e2e drives the compiled binary as a subprocess and
  asserts exit code, stdout, stderr, and file effects; arguments and stdin are
  validated at the edge; success is quiet, errors are explicit.
- **Language:** Rust (stable, pinned in `rust-toolchain.toml`). Composed from
  `languages/rust.md`: `rustfmt` + `clippy -D warnings` as strict gates,
  `cargo nextest` for the test runner, `cargo llvm-cov` enforcing the 95%
  coverage floor (lines, functions, regions) in the gate, and `cargo deny` +
  `cargo machete` as a dedicated supply-chain job. The release tier follows the
  reference's tag-driven `release-plz` + per-target native-runner archive model.
- **Cross-cutting:** `ci.md` (always) — CI runs `just bootstrap` then `just check`
  on a Linux/macOS/Windows matrix, with coverage and the dependency/security
  checks as their own jobs, and the live per-CLI harness checks plus benchmarks
  kept out of the gate as their own informational workflows.
- **Excluded — and why:** `monorepo.md` does not apply (one deliverable, one
  language — a single binary crate, not multiple apps/packages). `shapes/library.md`
  does not apply: this ships an executable, not a published library API (the
  intentionally public Rust surface in `lib.rs` exists only to test the engine,
  not as a distribution target). No intersection reference is pulled in because a
  `rust-cli.md` does not exist yet; its concerns (snapshot-testing a compiled
  binary, cross-platform release artifacts) are handled here directly via the
  `tests/e2e` suite and the release workflow.

## Layout

- `src/main.rs` — thin: parse args, dispatch, map the typed result to an exit
  code, print. No behavior here.
- `src/lib.rs` — wiring and the public API; `run()` is the entry point.
- `src/cli.rs` — clap definitions: args, subcommands, defaults in one place.
- `src/domain/` — pure engine (glob, rule, analyzer, decision). No I/O.
- `src/io/` — filesystem config discovery, harness stdin/stdout adapters, and the
  opt-in usage-history store.
- `src/commands/` — one module per CLI verb; orchestrates domain + io.
- `src/config.rs` — JSON rule schema and user/project merge (a boundary, not
  domain).
- `src/errors.rs` — typed errors.

## Hard rules

- Keep `domain/` free of filesystem, process, env, and terminal I/O. I/O stays
  at `io/` and `commands/` boundaries; never hide it in helpers that look pure.
- The engine never panics and never denies on a parse or internal error:
  unparseable or unsupported input defers.
- Typed errors via `thiserror`; surface them only at the app boundary. Validate
  config and inputs at load/parse time and return typed errors.
- `pub(crate)` by default. The intentional public API is `domain`, `config`,
  `errors`, and `run`.
- Prefer plain functions, structs, and enums; add traits only at real
  boundaries. No speculative abstractions, no `utils` grab-bags.

## Toolchain & dependencies

- `cargo` is the source of truth; `Cargo.lock` is committed (binary crate).
- Bump the pinned toolchain and dependencies deliberately via `just upgrade`,
  then re-run the gate and review the lockfile diff.
- Keep dependency features minimal; one tool per role. Run every task through
  `just`.
- `scripts/setup.sh` (`just setup`) is the one idempotent entry point for local
  setup; keep it re-runnable. `scripts/setup-check.sh` (`just setup-check`) stays
  a fast, install-free readiness check — resolved tools plus a fingerprint stamp
  (under `.dev/`) of the pinned versions, so it re-provisions after `just upgrade`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nickderobertis/allowlister](https://github.com/nickderobertis/allowlister) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-14 -->
