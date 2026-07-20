---
trigger: always_on
description: IMPORTANT: Never enter plan mode automatically!!! Never enter plan mode automatically!!!
---

# {{ project-name }}

IMPORTANT: Never enter plan mode automatically!!! Never enter plan mode automatically!!!

## Core Principles

### Completion Discipline

- **Do Not Stop Early**: If the user's requested outcome is not fully complete, do not stop at a draft, partial pass, or "good enough" result. Continue reviewing and improving until the request is genuinely handled or a concrete blocker requires user input.
- **Polish Bar**: Before declaring work complete, ask whether the result is fully polished, concrete, correct, complete, and elegant. If there is doubt, review the work again and update it.
- **Honest Status**: Do not claim a task is finished when it is only a first pass, scaffold, or partial draft. State the remaining gaps and keep working unless the user explicitly asks to pause.

### Code Quality

- **SOLID and DRY Principles**: Maintain clean, maintainable code following SOLID and DRY principles
- **No Incomplete Code**: Never write TODO comments or temporary solutions. If you encounter such a situation:
  1. Stop the current task
  2. Review the problem globally
  3. Rethink the design and identify the best alternative solutions
  4. Proceed with the complete solution
- **Thorough Analysis**: Always perform a comprehensive review and analysis of the problem before starting work
- Do not suppress dead code, remove them; Unless explicitly requested, do not go through deprecation process, just remove the code that is no longer needed.

### Development Workflow

- **Latest Dependencies**: Always search the web for the latest dependencies or helm charts or resources and their current usage patterns. If doing a deep research, put the research doc under ./docs/research. You shall look into that directory before doing researches.
- **Automation via Makefile**:
  - Explore existing Makefile targets and use them accordingly
  - For new automation tasks, always add a Makefile target instead of creating shell scripts
  - Keep automation consistent and discoverable

## Documentation

For specs, explore ./specs directory and put it to the right place, name the spec file as {feature-name}-{type}.md and update index.md accordingly. type can be prd, design, impl-plan, verification-plan, review, etc.

For docs, explore ./docs directory and put it to the right place, and update index.md accordingly. If you generate documentation that wasn't explicitly requested, make sure to place it under `./docs` and follow the same rule.

## Toolchain & Build

- Always use Rust 2024 edition with latest stable version. Pin version in `rust-toolchain.toml`.
- Verification must be scoped to the change, not run mechanically. Before finishing, inspect the diff and run the smallest meaningful checks that can catch regressions in the touched surface. Explain any skipped heavyweight gate.
- Run the full Rust gate set (`cargo build`, `cargo test`, `cargo +nightly fmt`, and `cargo clippy -- -D warnings`) when Rust source, public Rust APIs, tests, examples, build scripts, feature flags, workspace manifests, or generated Rust artifacts change.
- Use `cargo clippy -- -D warnings -W clippy::pedantic` for stricter linting on Rust code changes where it adds signal. Allow specific lints with justification.
- Run `cargo audit` and `cargo deny check` when dependencies, lockfiles, license policy, supply-chain configuration, or release packaging change. Otherwise run them periodically, not for unrelated documentation edits.
- For documentation/spec/skill-only changes that do not alter Rust code, APIs, Cargo manifests, examples/doctests, generated artifacts, or release packaging, do not run Rust build/test/clippy. Validate the changed artifacts instead: proofread rendered Markdown as needed, check touched links/indexes, run `make check-agent-sync` for AGENTS/CLAUDE/skill edits, and run skill validation when skill folders change.
- If unsure whether code behavior is affected, choose the narrowest Rust command that answers the question first (for example `cargo test -p <crate> <test>` or `cargo check -p <crate>`) and broaden only when the result warrants it.
- Enable all rustc lints in Cargo.toml: `#![warn(rust_2024_compatibility, missing_docs, missing_debug_implementations)]`.
- DO NOT use `cargo clean` at any time. If you indeed need it, ask user for permission

## Error Handling

- Never use `unwrap()` or `expect()` in production code. Always handle errors properly with `?` operator or explicit match.
- Use `thiserror` for library error types (with custom error enums). Use `anyhow` for application error handling.
- Implement proper error context with `.context()` or `.with_context()` when propagating errors.
- Use `Result<T>` as return type for fallible functions. Never use `Option` to represent errors.
- For unrecoverable errors in applications, use `panic!`. For libraries, always return `Result`.
- Define domain-specific error types using enums with `thiserror`. Include source errors with `#[source]`.

## Async & Concurrency

- Use Tokio as async runtime. Always specify features explicitly (e.g., `tokio = { version = "1", features = ["rt-multi-thread", "macros"] }`).
- Prefer message passing (channels) over shared state. Use `tokio::sync::mpsc` for MPSC, `flume` for faster channels.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tyrchen/rust-lib-template](https://github.com/tyrchen/rust-lib-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
