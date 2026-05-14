---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

`assert-struct` is a procedural macro library for ergonomic structural assertions in Rust tests. It enables deep, partial matching of complex data structures without manually referencing every field - particularly useful for testing typed JSON responses and other nested data structures.

**Important:** This is a new, experimental crate in early development. We are NOT concerned with:
- Backward compatibility (we can make breaking changes freely)
- Migration guides (no existing users to migrate)
- Performance benchmarks (premature optimization)
- Semantic versioning constraints

Focus on clean design, good architecture, and comprehensive functionality rather than compatibility concerns.

### Design Philosophy

1. **Simplicity First**: Resist adding features just because they're possible. Ship minimal valuable functionality, then expand based on real user needs.
2. **No Speculative Features**: Don't implement "nice to have" features without clear use cases. Examples of avoided complexity:
   - Utility types (CaseInsensitive, Prefix, Suffix) - users can implement these if needed
   - Complex pattern combinators (Vec for OR, tuples for AND) - adds complexity without proven value
   - Option as wildcard pattern (using None to mean "any value") - clever but unnecessary
3. **Performance Optimization**: Only optimize the common case (string literals compiled at macro expansion time), not every possible case.

### Core Features (Implemented)
- **Partial matching**: Check only the fields you care about with `..`
- **Nested struct support**: Deep assertions without verbose field access chains
- **Nested field access**: Direct access to nested fields `outer.inner.field: value`
- **Comparison operators**: `<`, `<=`, `>`, `>=` for numeric assertions
- **Equality operators**: `==`, `!=` for explicit equality checks
- **Range patterns**: `18..=65`, `0.0..100.0` for range matching
- **Regex patterns**: `=~ r"pattern"` for string matching (feature-gated)
- **Like trait**: `=~ expr` for flexible pattern matching with variables/expressions
- **Slice patterns**: Element-wise patterns for Vec fields `[> 0, < 10, == 5]`
- **Set patterns**: Unordered collection matching `#(1, 2, 3)` or `#(> 0, < 10, ..)`
- **Index operations**: Direct indexing into collections `values[0]: 10`, `matrix[0][1]: 2`
- **Enum support**: Full support for Option, Result, and custom enums (all variant types)
- **Tuple support**: Multi-field tuples with advanced patterns `(> 10, < 30)`
- **Method call patterns**: `field.method(): value` and `(0.method(): value, _)` for tuple elements
- **Pattern composition**: Combine all features (e.g., `Some(> 30)`, `Event::Click(>= 0, < 100)`)
- **Anonymous struct patterns**: Use bare `{ field: value }` to avoid imports (always non-exhaustive, `..` never required)
- **Smart pointer dereferencing**: Direct pattern matching through `Box`, `Arc`, `Rc` with `*field: value`

### Improved Error Messages
- **Fancy error formatting**: Shows pattern context with precise failure location
- **Pattern underlining**: Exact position of mismatch highlighted with `^^^^^`
- **Field path tracking**: Full path to failing field (e.g., `user.profile.age`)
- **Equality vs comparison**: Different formatting for `==` patterns showing expected value
- **Zero runtime cost**: Pattern strings generated at compile time

### Example Use Case
```rust
// Instead of:
assert_eq!(response.user.profile.settings.notifications.email, true);
assert!(response.user.profile.age >= 18);
assert!(response.items.len() > 0);

// You can write:
assert_struct!(response, Response {
    user: User {
        profile: Profile {
            age: >= 18,
            ..
        },
        ..
    },
    items.len(): > 0,        // Method call patterns
    items: [> 0, < 100, > 0],  // Element-wise patterns
    items[0]: > 0,           // Index operations
    ..
});

// With improved errors showing:
assert_struct! failed:

   | Response { ... Profile {
comparison mismatch:
  --> `response.user.profile.age` (line 10)
   |         age: >= 18,
   |              ^^^^^ actual: 17
   | } ... }
```

## Commit and PR Conventions

This repository follows the [Conventional Commits](https://www.conventionalcommits.org/) specification. All commit messages and PR titles must use the format: `type: description` (e.g., `feat: add new pattern`, `fix: resolve parsing bug`).

Allowed types: `feat`, `fix`, `docs`, `style`, `refactor`, `perf`, `test`, `build`, `ci`, `chore`, `revert`. Scopes are optional. The subject must start with a lowercase letter.

## Development Commands

### Build & Test
```bash
cargo build                    # Build in debug mode
cargo test                     # Run all tests
cargo test --no-default-features  # Test without regex feature (IMPORTANT for CI)
cargo test -- --nocapture     # Run tests with println! output
cargo test <test_name>        # Run specific test
cargo test --doc              # Run documentation tests
```

### Code Quality
```bash
cargo fmt                                # Format code
cargo fmt -- --check                    # Check formatting

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [carllerche/assert-struct](https://github.com/carllerche/assert-struct) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
