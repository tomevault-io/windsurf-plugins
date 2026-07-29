---
trigger: always_on
description: Source of truth for coding standards and patterns. Describes approaches
---

# distant Coding Conventions

Source of truth for coding standards and patterns. Describes approaches
abstractly — the codebase is the source of truth for current names and paths.

## Module Organization

- Library crates re-export public items at the crate root for flat imports
- Avoid deeply nested public paths — users import directly from the crate root
- Private inner modules with selective `pub use` re-exports
- Group related functionality into focused modules
- Use `mod.rs` or named module files consistently within each crate

## Abstraction Philosophy

- Prefer trait hierarchies for swappable implementations over concrete types
- Separate complex logic into dedicated structs — CLI commands translate user
  options, then delegate to the struct for actual work
- Use newtype wrappers for domain concepts (paths, IDs, keys) to prevent
  misuse at the type level
- Use builder patterns for types with many configuration options
- Use type aliases to simplify complex generic signatures and make intent clear
- Avoid premature abstraction — three similar lines is better than a one-use
  helper function

## Error Handling

- **Binary crate**: Use flexible error chaining (anyhow-style) with contextual
  messages. Every error propagation adds context describing what operation failed:
  ```rust
  let config = load_config().context("Failed to load configuration file")?;
  ```
- **Library crates**: Define situation-specific error types using derive macros
  (derive_more for Display, Error, From). No anyhow in library code:
  ```rust
  #[derive(Debug, Display, Error, From)]
  pub enum MyError {
      #[display("operation failed: {}", _0)]
      OperationFailed(#[error(not(source))] String),
  }
  ```
- Use type aliases to simplify Result types: `pub type MyResult = Result<(), MyError>;`
- Custom error types should be enums with descriptive variants
- Error messages should be helpful and actionable for end users
- No `unwrap()` in production code except where safety is documented with a
  comment explaining why it cannot fail

## Documentation Standards

- All public items in library crates MUST have doc comments
- Start with a one-line summary explaining what the item does
- Include `# Examples` section with compilable doctests for non-trivial items
- Include `# Errors` section for fallible functions
- Show typical usage and edge cases in examples
- Explain *why* to use something, not just *how*
- Do not add doc comments to `#[cfg(test)] mod tests` blocks

### Documentation Example Pattern

Good documentation follows this structure:

```rust
/// One-line summary of what this type/function does.
///
/// Longer description if needed, explaining behavior, constraints,
/// and design rationale.
///
/// # Examples
///
/// ```
/// // Show typical usage with realistic values
/// let result = my_function("example input");
/// assert_eq!(result, expected_output);
/// ```
///
/// # Errors
///
/// Returns `Err` if [describe the failure condition].
```

### Module Documentation Style

Module-level doc comments (`//!`) should describe what the module provides
and when to use it — not how it was created or where code was extracted from.
Follow the Rust standard library style (e.g., `std::fs`, `std::net`).

Avoid:
- "Extracted from ..." or "Refactored out of ..."
- References to specific orchestrators or callers
- Implementation history

Prefer:
- What functionality the module provides
- When/why to use it
- Key types and functions available

## Async Patterns

- Use Tokio with full features as the async runtime
- Async trait methods return pinned futures for dynamic dispatch
- Use lazy async initialization for expensive one-time setup
- Avoid blocking calls in async context — use spawn_blocking if needed
- Prefer `async fn` where the compiler allows; use pinned futures in trait
  definitions
- Keep async blocks small — if an async block passed to a combinator (e.g.,
  `get_or_try_init`) exceeds ~10 lines, extract it into a named async method

## Naming

- Rely on module namespaces for clarity — don't redundantly prefix type names
  with the crate or module name (e.g., `Client` not `DistantClient`)
- Provide clear, meaningful names that follow established Rust ecosystem norms
- Prefix unused bindings with `_` (including phantom type markers)
- CLI options: long form for less-used options, short flags for frequent ones,
  concise documentation per option

## Import Ordering

1. `std::` imports
2. External crate imports (alphabetical)
3. Internal `crate::` imports
- Import types used in signatures and pattern matches — avoid inline
  `crate::module::Type` references
- For functions: import the parent module (`use crate::utils;`) and call
  via `utils::func()`. Do not import functions directly unless they are
  used frequently in a tight scope (e.g., test helpers).

## Inline Comments

- Always include a blank line above a line comment unless the comment is
  at the start of a function/block or inside a nested block (if/match arm)
- Don't reference private functions in doc comments — describe the behavior
  instead
- Remove comments that restate what the test name or assertion message
  already communicates

## Serialization

- JSON and TOML for human-readable config
- MessagePack for binary protocol

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chipsenkbeil/distant](https://github.com/chipsenkbeil/distant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
