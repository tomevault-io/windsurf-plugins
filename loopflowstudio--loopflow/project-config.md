---
trigger: always_on
description: This is the governing document of the loopflow codebase. Humans and LLMs alike are expected to follow it.
---

# Loopflow Style Guide

This is the governing document of the loopflow codebase. Humans and LLMs alike are expected to follow it.

## Quick Reference

**Python:**
- Use `uv run` or activate `.venv` before any Python command
- Prefix private functions with `_`
- Return `None` for "not found"; raise exceptions for "shouldn't happen"
- No `Args:`/`Returns:` docstrings—if types are clear, skip the docstring

**Rust:**
- Run `cargo fmt` and `cargo clippy` before committing
- Return `Option<T>` for "not found"; return `Result<T, E>` for failures
- Use `expect("reason")` over `unwrap()` outside tests
- Derive `Debug` on all public types

**Both:**
- Mock side effects, but don't test mock wiring or reshape production code for tests
- Design docs go under `scratch/`; `lf op pr land` removes `scratch/*` contents
- Auto runs are headless: make executive decisions and keep moving, note genuinely ambiguous choices in `scratch/questions.md`

## File-Type Guidelines

When editing `*.py` files:
- Put imports at the top, not inline
- Use type hints on all public functions
- One-line docstring if any; skip if the name and types are clear

When editing `*_test.py` or `test_*.py` files:
- Keep tests short and focused on one behavior
- Mock side effects (network, subprocess), but assert on results, not mock calls
- Delete flaky tests rather than adding retries

When writing CLI code with Typer:
- Prefer lowercase short flags (`-p`, `-c`), support uppercase as aliases
- Pass args through to underlying tools rather than re-implementing
- Default to sensible behavior (e.g., whole repo as context)

When editing `README.md` files:
- Examples first, explanation after—show `lf debug -c`, then say what it does
- Action-focused tables: "What it does" not "What it is"
- Terse prose around code blocks—the code speaks
- One good example beats three similar ones
- No preamble: "Assembles context" not "Loopflow assembles context for you"
- Write for users, not maintainers
- Update when adding or changing user-facing features

When editing docs in `scratch/`:
- Focus on what's left to build, not what's done
- `lf review` writes its assessment under `scratch/`
- `lf op pr land` removes `scratch/*` contents automatically

When editing `*.rs` files:
- Run `cargo fmt` before committing; CI enforces it
- Run `cargo clippy -- -D warnings` locally; CI treats warnings as errors
- Dead code must be deleted, not commented out (use git for history)
- If code is intentionally unused (e.g., for FFI/PyO3), use `#[allow(dead_code)]` with a comment explaining why
- Avoid `use super::*` in submodules; use explicit imports so dependencies between modules are visible
- Derive `Debug` on all public types; add `Clone`, `PartialEq`, `Default` where sensible
- Use `thiserror` for library error types callers need to match on
- Use `expect("why this is safe")` over `unwrap()` outside tests
- Conversion methods: `as_` (cheap/borrowed), `to_` (allocates), `into_` (consumes self)
- No `get_` prefix on getters: `fn name(&self)` not `fn get_name(&self)`
- Return `Option<T>` for "not found", `Result<T, E>` for "something went wrong"
- Newtypes for domain concepts: `struct RunId(String)` not `type RunId = String`
- Every `unsafe` block requires a `// SAFETY:` comment explaining invariants
- When a name conflicts with a keyword: use `r#type` or `type_`, not `typ`
- Use `#[non_exhaustive]` on public enums that may grow
- Never use `()` as an error type
- For public APIs, include `# Panics`/`# Errors`/`# Safety` doc sections where non-obvious

When editing Rust tests:
- `unwrap()` is fine in tests
- Use `#[test]` for unit tests in the same file
- Integration tests go in `tests/` directory
- Mock via closures or `#[cfg(test)]`, not factory traits or extra abstractions

# Goals

## Clarity

Design around data structures and public APIs. Aim for a 1:1 mapping between real-world concepts and their representation in code.

Write code that demonstrates its own correctness. If a feature exists, write a test that proves it works. Assume you won't finish everything you start—make it easy to see what's done and what's broken.

## Simplicity

Every line of code must earn its place. Readable code is not terse code; don't sacrifice clarity for brevity. But recognize that lines can be net-negative:

* Unused code
* Comments that restate the obvious
* Checks for impossible conditions

Start with minimal data structures and APIs. If the core is right, trimming excess at the edges is straightforward.

# Development Environment

Use `uv` for Python package management. Never use pip directly.

```bash
# Python
uv sync                       # Install dependencies
uv run pytest python/tests/   # Run Python tests
uv run lf agent --help        # Run commands

# Or activate the venv
source .venv/bin/activate
pytest python/tests/

# Rust
cargo build                   # Build all crates
cargo test                    # Run all Rust tests
cargo fmt                     # Format code
cargo clippy -- -D warnings   # Lint (warnings = errors)
```

See TESTING.md for the full test suite (Python, Swift, Rust, Concerto UI). CI runs all.

# Code Organization

Follow PEP8. Consistency with existing code matters more than any specific rule.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [loopflowstudio/loopflow](https://github.com/loopflowstudio/loopflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-26 -->
