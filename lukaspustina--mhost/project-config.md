---
trigger: always_on
description: - Do NOT add a `Co-Authored-By` line for Claude in commit messages.
---

# CLAUDE.md — mhost

## Rules

- Do NOT add a `Co-Authored-By` line for Claude in commit messages.
- Don't add heavy dependencies for minor convenience — check if existing deps already cover the need.
- Don't mix formatting-only changes with functional changes in the same commit.
- Don't modify unrelated modules "while you're in there" — keep changes scoped.
- Don't add speculative flags, config options, or abstractions without a current caller.
- Don't bypass failing checks (`--no-verify`, `#[allow(...)]`) without explaining why.
- Don't hide behavior changes inside refactor commits — separate them.
- Don't include PII, real email addresses, or real domains (other than example.com) in test data, docs, or commits.
- If uncertain about an implementation detail, leave a concrete `TODO("reason")` rather than a hidden guess.
- Don't change the `-p` short flag mapping — it intentionally means `--predefined` globally and `--show-partial-results` in subcommands. This is accepted CLI design.

## Engineering Principles

- **Performance**: Prioritize efficient algorithms and data structures. Benchmark critical paths, avoid unnecessary allocations and copies.
- **Efficiency**: Make use of mhost async, multi-query capabilities to parallelize lookups across multiple nameservers and record types simultaneously.
- **Rust patterns**: Use idiomatic Rust constructs (enums, traits, iterators) for clarity and safety. Leverage type system to prevent invalid states.
- **KISS**: Simplest solution that works. Three similar lines beat a premature abstraction.
- **YAGNI**: Don't build for hypothetical future requirements — solve the current problem.
- **DRY + Rule of Three**: Tolerate duplication until the third occurrence, then extract.
- **SRP**: Each module/struct has one reason to change. Split when responsibilities diverge.
- **Fail Fast**: Validate at boundaries, return errors early, don't silently swallow failures.
- **Secure by Default**: Sanitize external input, no PII in logs, prefer safe APIs.
- **Determinism**: Same input → same output. Pin randomness in tests, avoid time-dependent logic where possible.
- **Reversibility**: Prefer changes that are easy to undo. Feature flags over big-bang migrations, small commits over monolithic ones.

## Project Overview

**mhost** is a modern DNS lookup utility and reusable Rust library — an advanced replacement for `host`/`dig`. Two binaries: `mhost` (CLI) and `mdive` (interactive TUI).

Feature flags (layered):
- **`app-lib`** — shared app layer (`app/common/`): lints, discovery strategies, rendering, styles. Pulls in `anyhow`, `humantime`, `services`.
- **`app-cli`** (default) — CLI binary (`mhost`). Depends on `app-lib`. Adds `clap`, `tabwriter`, `tracing-subscriber`, etc.
- **`app-tui`** — TUI binary (`mdive`). Depends on `app-lib`. Adds `ratatui`, `crossterm`, `regex`.
- **`app`** / **`tui`** — backward-compatibility aliases for `app-cli` / `app-tui`.

Without any `app-*` feature: library-only build with minimal dependencies.

- **Author**: Lukas Pustina | **License**: MIT / Apache-2.0 | **Edition**: 2021
- **Repository**: https://github.com/lukaspustina/mhost.git

mhost provides and all functionality must adhere to these core principles:

- high performance
- high efficiency
- high stability
- high security

## Roadmap

The roadmap of this project is in file ROADMAP.md. It contains a prioritized list of features, improvements, and bug fixes planned for future releases. The roadmap is a living document and may be updated as the project evolves.

## Build & Test

```sh
cargo build                        # Build everything (default feature = "app-cli")
cargo build --lib                  # Build library only
cargo build --features app-tui     # Build with TUI (mdive binary)
cargo check                        # Type-check without full compilation
cargo test --lib                   # Unit tests (fast, no network needed)
cargo test                         # All tests incl. CLI integration tests (slower, needs network)
cargo clippy                       # Lint
cargo fmt                          # Format
cargo run --bin mdive --features app-tui -- example.com  # Run mdive
```

### Test guidelines

- **`cargo test --lib`** is the reliable quick check — no network needed.
- **`cargo test`** also runs lit-based CLI integration tests (`tests/cli_output_tests.rs`) that make real DNS queries via `8.8.8.8`. These may fail due to DNS timeouts or changed records.
- **Every new rdata type or RecordType variant must have unit tests.** Each rdata module has a `#[cfg(test)] mod tests` block covering constructor/accessor round-trips and any enum conversions (`From<u8>`, `Display`).
- **`RecordType::from_str` must cover all variants.** If you add a new `RecordType` variant, add it to `FromStr`, `all()`, and the `from_str_all_standard_types` test. The `display_round_trip` test will catch omissions.
- **`RData` accessor tests** in `src/resources/rdata/mod.rs` verify each variant's accessor returns `Some` and unrelated accessors return `None`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lukaspustina/mhost](https://github.com/lukaspustina/mhost) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
