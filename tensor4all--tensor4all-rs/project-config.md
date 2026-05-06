---
trigger: always_on
description: Read `README.md` and `REPOSITORY_RULES.md` before starting work.
---

# Agent Guidelines for tensor4all-rs

Read `README.md` and `REPOSITORY_RULES.md` before starting work.

## Development Stage

**Early development** - no backward compatibility required. Remove deprecated code immediately.

## General Guidelines

- Use same language as past conversations (Japanese if previous was Japanese)
- Source code and docs in English
- Each crate in `crates/` is independent with own `Cargo.toml`, `src/`, `tests/`
- **Bug fixing**: When a bug is discovered, always check related files for similar bugs and propose to the user to inspect them

### API Reference (Check First)

```bash
cargo run -p api-dump --release -- . -o docs/api
```

Read `docs/api/*.md` before source files. Only read source when API doc is insufficient.

## Context-Efficient Exploration

- Use Task tool with `subagent_type=Explore` for open-ended exploration
- Use Grep for structure: `pub fn`, `impl.*for`, `^pub (struct|enum|type)`
- Read specific lines with `offset`/`limit` parameters
- Prefer API docs over full source files

## Code Style

`cargo fmt` for formatting, `cargo clippy` for linting. Avoid `unwrap()`/`expect()` in library code.

**Always run `cargo fmt --all` before committing changes.**

## Documentation Requirements

### Rustdoc Standards

Every public type, trait, and function **must** have doc comments with the following:

**Types (struct/enum/trait):**
- Summary: what it represents, when to use it (1-2 sentences)
- Related types: relationship to similar types (e.g., "`TensorTrain` is the simple chain version; `TreeTN` is the general tree version")
- `# Examples` section with runnable code and assertions

**Functions/methods:**
- Summary: what it does (1 sentence)
- Arguments: meaning, constraints, typical values for each parameter (especially for `Options` types)
- Returns: what is returned, how to use it
- `# Panics` or `# Errors`: under what conditions it fails
- `# Examples` section with runnable code and assertions

**Options/Config types (critical for usability):**
- Each field: meaning, recommended values, and default behavior
- Field relationships and trade-offs (e.g., `rtol` vs `max_bond_dim`)
- "When in doubt" defaults

### Code Example Rules

- All doc examples **must** be runnable (`ignore` and `no_run` attributes are **prohibited**)
- All doc examples **must** include assertions verifying correctness (not just compilation/execution)
  - Use `assert!`, `assert_eq!`, `approx::assert_abs_diff_eq!`, etc.
- mdBook guide code blocks follow the same rules: runnable with assertions
- mdBook code blocks use hidden lines (`# ` prefix) for `use` statements and `fn main()` wrappers

### CI Verification

- `cargo test --doc --release --workspace` must pass (rustdoc examples)
- `./scripts/test-mdbook.sh` must pass (mdBook guide examples; raw `mdbook test docs/book` does not receive the resolved `--extern` flags that the guide snippets need)

### Public Surface Drift

- `README.md`, rustdoc, and examples must not claim more than the current public surface actually provides.
- When changing public APIs, documented capabilities, or user-facing examples, check for stale names, stale capability claims, and references to removed paths or workflows.
- Keep documentation slightly behind reality if validation is incomplete; do not advertise partially landed surfaces as stable or fully supported.

### Online Tutorial Synchronization

- The live online tutorials are in `docs/book/src/tutorials/`.
- Runnable tutorial demos live in `docs/tutorial-code/src/bin/` with shared helpers in `docs/tutorial-code/src/`.
- When changing public APIs, tutorial code, generated tutorial CSV/PNG artifacts, or examples quoted by the online tutorials, update the live mdBook tutorial page in the same branch.
- Treat `docs/tutorial-code/docs/tutorials/` as legacy/reference material unless this policy is changed explicitly.

## Error Handling

- `anyhow` for internal error handling and context
- `thiserror` for public API error types

### C API Error Handling (Known Issue)

**IMPORTANT**: The C API (`tensor4all-capi`) currently discards all error details at the FFI boundary. See [#228](https://github.com/tensor4all/tensor4all-rs/issues/228).

- `catch_unwind` catches panics but the panic message is lost (`result.unwrap_or(T4A_INTERNAL_ERROR)`)
- `Err(e)` variants are discarded (`Err(_) => T4A_INTERNAL_ERROR`)
- FFI consumers (Julia, Python) only see a generic "Internal error" with no diagnostic info
- **~76 `catch_unwind` sites** and **~47 `Err(_)` discard sites** across the capi crate need updating
- **Do not add new `catch_unwind` / `Err(_) => T4A_INTERNAL_ERROR` patterns** without preserving error messages
- When #228 is resolved, use the `t4a_last_error_message` API and `run_catching` helper for all new FFI functions

## Testing

```bash
cargo nextest run --release --workspace          # Full suite
cargo nextest run --release --test test_name     # Specific test
cargo nextest run --release -p crate_name        # Single crate
```

**Always use `--release` mode for tests** to avoid excessive execution time in debug builds.

- Private functions: `#[cfg(test)]` module in source file
- Integration tests: `tests/` directory

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tensor4all/tensor4all-rs](https://github.com/tensor4all/tensor4all-rs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
