---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Branching

- `master` is a **release-only** branch, updated every few months. Do not use it as a base for diffs or PRs during active development.
- `v1.0-dev` is the current active development branch. Use it as the base for general diffs, comparisons, and new feature branches.
- PR and commits should follow conventional commit naming rules.

## Build Commands

```bash
cargo build                    # Debug build
cargo build --release          # Release build
cargo run                      # Run application
cargo fmt --all                # Format code
cargo clippy --all-features --all-targets -- -D warnings  # Lint (warnings as errors)
```

## Testing

```bash
cargo test --all-features --workspace              # All tests
cargo test --doc --all-features --workspace        # Doc tests only
cargo test <test_name> --all-features              # Single test
cargo test --test kittest --all-features           # UI integration tests (egui_kittest)
cargo test --test e2e --all-features               # End-to-end tests
```

### Backend E2E tests (network-dependent)

Tests that exercise backend tasks against a live Dash testnet via SPV (no GUI). Marked `#[ignore]` — require network access, a funded wallet, and serial execution. See `tests/backend-e2e/README.md` for run commands, architecture, and writing guide.

Test locations:
- Unit tests: inline in source files (`#[test]`)
- UI integration: `tests/kittest/`
- E2E: `tests/e2e/`
- Backend E2E: `tests/backend-e2e/` (network-dependent, `#[ignore]`)

Always run `cargo clippy` and `cargo +nightly fmt` when finalizing your work.

### User stories catalog

When a PR adds or significantly changes user-facing features, check `docs/user-stories.md`:
- If a new feature matches no existing story, add one following the existing format (ID, persona, description, acceptance criteria, `[Implemented]` tag).
- If a `[Gap]` story is now implemented, flip its tag to `[Implemented]`.
- Skip user-story updates for non-functional changes (CI, docs, formatting, refactoring).

## CI: Safe Cargo Wrapper

In GitHub Actions (Claude Code workflow), use `scripts/safe-cargo.sh` instead of `cargo` directly. This wrapper strips CI secrets from the environment before running cargo, preventing build scripts from accessing credentials.

```bash
scripts/safe-cargo.sh build --all-features
scripts/safe-cargo.sh test --all-features --workspace
scripts/safe-cargo.sh clippy --all-features --all-targets -- -D warnings
scripts/safe-cargo.sh +nightly fmt --all
```


## Coding Conventions

### General rules

* When a method takes `&AppContext` (or `Option<&AppContext>`), place it as the first parameter after `self`.
* Screen constructors handle errors internally via `MessageBanner` and return `Self` with degraded state. Keep `create_screen()` clean — no error handling at callsites.
* **i18n-ready strings**: All user-facing strings (labels, messages, tooltips, errors) must be simple, complete sentences. Avoid concatenating fragments, positional assumptions, or grammar that breaks in other languages. Each string should be extractable as a single translation unit with named placeholders for dynamic values and no logic in the text itself. Current code uses standard Rust format specifiers (`{name}`, `{max}`). When i18n extraction happens later, these will become Fluent-style placeholders (`{ $name }`, `{ $max }`).
* **Never parse error strings** to extract information. Always use the typed error chain (downcast, match on variants, access structured fields). If no typed variant exists for the information you need, define a new `TaskError` variant or extend the existing error type. String parsing is fragile, breaks on message changes, and bypasses the type system.
* **Validation placement**: Pure input validation (format, length, character sets) lives in `model/` as stateless functions — single source of truth, unit-testable, no dependencies on `AppContext` or `Sdk`. Backend tasks are the authoritative enforcement layer: they call model validators for format checks AND perform stateful validation that requires network or database (existence checks, uniqueness, business rules). UI screens may call model validators for instant user feedback, but must never implement their own validation logic — always delegate to the model function.

### Error messages

User-facing error messages (shown in `MessageBanner` via `Display`) must follow these rules:

1. **Audience**: Write for the Everyday User persona (`docs/personas/everyday-user.md`). No jargon — no "consensus error", "nonce", "state transition", "SDK", "RPC", or error codes.
2. **Structure**: *What happened* + *what to do*. Every message must include a concrete action the user can take themselves: retry, wait, try a different approach. Never redirect to "contact support" — users must be able to self-resolve.
3. **Tone**: Calm, direct, brief. Not apologetic ("Sorry!"), not alarming ("Something went wrong!"), not vague ("An error occurred").

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dashpay/dash-evo-tool](https://github.com/dashpay/dash-evo-tool) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-04 -->
