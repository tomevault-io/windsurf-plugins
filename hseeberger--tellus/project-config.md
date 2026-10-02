---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

Tasks are defined in the [justfile](justfile):

- `just all`: check, fmt, lint, test and doc; the full local gate for every crate except `tellus-comparison`. The `tellus-persistence-postgres` tests use testcontainers, so the gate needs Docker.
- `just check` / `just lint` / `just test`: the individual steps, each a feature matrix, every run scoped to one crate. Check and lint drive the matrix with `cargo hack --feature-powerset`: the `powerset` variable in the justfile takes `hotpath` and `hotpath-alloc` off the axes, which leaves ten runs of `-p tellus`, plus one `hotpath` run, one `--all-features` run (cargo-hack drops its own once either of those flags is given) and one run of `-p tellus-persistence-postgres`; adding a feature therefore extends check and lint by itself. No feature is declared mutually exclusive with another, and none may be: cargo-hack resolves a group's features, so excluding `persistence` against `persistence-tests` also drops `persistence-tests,persistence-in-memory`, since both resolve to include `persistence`, and that pair is the one combination which compiles `tests/persistence_tests.rs` to something. Implications need no group anyway, since cargo-hack drops a combination whose features resolve to the same set. Test stays hand-picked, since its combinations cost test execution rather than a compile: six runs of `-p tellus` (no features, `serde`, `persistence`, `persistence-in-memory`, `persistence-tests` with `persistence-in-memory`, all features) plus one of `-p tellus-persistence-postgres`. No run spans the workspace: a build containing both crates feature-unifies `tellus` with `persistence` enabled, so only scoped runs exercise the reduced-feature configurations, and only a scoped run builds `tellus-persistence-postgres` against the feature set it actually declares (`cargo hack` keeps that property: it is invoked per crate, never with `--workspace`). `just doc` runs workspace-wide with all features.
- `just fmt`: formats Rust (nightly rustfmt, the justfile derives the matching nightly from the installed stable) and TOML (taplo). Plain `cargo fmt` is not enough; the rustfmt config uses unstable options.
- Single test: `cargo test -p tellus --test watch <test_name>`; a persistence test needs the features its file is gated on, or it silently runs none (`--features persistence-in-memory` for `persistence.rs`, `host_persistence.rs` and `persistence_tests.rs`, which also needs `persistence-tests`) (integration tests live in `tellus/tests/`: `ask.rs`, `persistence.rs`, `persistence_tests.rs`, `supervision.rs`, `termination.rs`, `watch.rs`).
- Examples: `just run-examples-hello`, `just run-examples-scatter-gather`, `just run-examples-event-sourced-counter`.

Benchmarks:

- `just profile` / `just profile-alloc`: run `tellus/examples/profile.rs` with hotpath profiling, reporting per-function timings or allocation bytes for the instrumented hot path (send path, mailbox, run loop, termination). Instrumentation is gated behind the off-by-default `hotpath` feature; read the report as relative attribution, criterion stays the source of truth for absolute regressions.
- `just profile-alloc-gate`: run the profiling workload with allocation tracking and fail unless `tell`, `reserve` and `receive_incoming` allocate exactly 0 bytes; `profile-alloc-check <file>` applies that check to an existing JSON report. CI (`profile.yml`) profiles every PR against its merge base, posts the per-function comparison as an informational PR comment, and fails the build only on the zero-alloc check.
- `just profile-persistence` / `just profile-persistence-alloc`: the same for the persistence code (`tellus/examples/profile_persistence.rs`, features `hotpath` plus `persistence` and `persistence-in-memory`): command settlement (encode, append, apply, snapshot) and recovery (snapshot load, paged read, replay) against an in-memory store. Settlement allocates by design (payload buffers, manifests), so most of the alloc report is a budget, not a zero gate; the exceptions are `apply_events` and `replay_page`, which must stay allocation-free: `just profile-persistence-alloc-gate` runs the workload and fails on a violation, `profile-persistence-alloc-check <file>` applies that check to an existing JSON report, and CI enforces it alongside the messaging zero-alloc check and includes the comparison in the profile comment.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hseeberger/tellus](https://github.com/hseeberger/tellus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
