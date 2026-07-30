---
trigger: always_on
description: - Brevity is a component of quality. Keep code lean and
---

# Development Conventions

## Coding Style

### General Principles

- Brevity is a component of quality. Keep code lean and
  complete; no bloat.
- Small, composable, single-purpose functions are the
  default unit of organization. Split code into small
  files with focused responsibilities.
- Minimize side effects. Prefer pure transformations when
  feasible: data in, data out. Resist mutable state when
  feasible and outside the critical paths.
- Keep functions short enough to reason about in isolation.

### Important Tools

- **Clippy**: Enforce idiomatic Rust and catch common mistakes
- **rustfmt**: Ensure consistent code formatting
- **cargo-audit**: Check for vulnerable dependencies
- **cargo-deny**: Enforce supply chain safety policies
- **cargo-machete**: Detect unused dependencies
- **cargo-semver-checks**: Lint for SemVer violations
- **rustdoc**: Generate the API documentation
- **cargo xtask**: Developer task runner for benchmarks, flamegraphs, and debug utilities
- **benchmarks**: Criterion microbenchmarks and scenario-based load tests ([Fortio], [Vegeta])

[Fortio]: https://github.com/fortio/fortio
[Vegeta]: https://github.com/tsenart/vegeta

### Comments vs Tracing

Comments answer **"why?"**, never **"what?"**.

**"What?" belongs in `tracing`**, not comments. If a
comment describes what the code is doing at runtime
("parse the config", "reject the request", "skip this
filter"), replace it with a `tracing::debug!`,
`tracing::trace!`, or `tracing::info!` call. Runtime
narration (what the code did, what it decided, what it
skipped) is structured logging, not commentary.

**"Why?" belongs in comments**, but only when
non-obvious. A hidden constraint, a subtle invariant, a
workaround for a specific bug, or behavior that would
surprise a reader: these justify a comment. If removing
the comment would not confuse a future reader, do not
write it.

**"What?" at the code level needs neither.** Well-named
identifiers already explain what the code does. Do not
write comments that restate what names already convey.

### Testing

**New capabilities require all of the following:**

1. Unit tests covering the implementation
2. Integration tests proving end-to-end behavior
3. An example config in `examples/configs/`
4. A functional integration test for the example config
   in `tests/integration/tests/suite/examples/`
5. Run `cargo xtask sync-example-readme --fix` to
   regenerate `examples/README.md`
6. Significant changes need to be [benchmarked].

This is not optional. A feature without tests and an
example is not complete.

Prefer more doctests when in doubt. Duplicative coverage
between doctests and unit/integration tests is fine.

Prefer assertion messages over inline comments. Put the
explanation in the assertion's message argument so it
prints on failure:

```rust
// Bad:
// ACL should block loopback
assert_eq!(status, 403);

// Good:
assert_eq!(status, 403, "ACL should block loopback");
```

[benchmarked]:../benchmarks.md

### RFC Conformance

When implementing protocol-level behavior (HTTP semantics,
header handling, TLS, etc.), identify the governing RFCs
and verify conformance against them.

- Cite the specific RFC number and section in test names
  or doc comments for protocol conformance tests.
- RFC references in doc comments must use reference-style
  rustdoc links to the IETF datatracker:
  ```rust
  /// Safe methods per [RFC 9110 Section 9.2.1].
  ///
  /// [RFC 9110 Section 9.2.1]: https://datatracker.ietf.org/doc/html/rfc9110#section-9.2.1
  ```
- When in doubt about an edge case, the RFC is the
  authority, not other proxy implementations.
- Add dedicated conformance tests when implementing
  RFC-specified behavior. These live in
  `tests/conformance/`.

See also [HTTP Correctness](../architecture/http-correctness.md)
for what Praxis enforces vs what Pingora handles.

### Rules, Practices & Lints

Security is enforced at the lint level. See lints in
[Cargo.toml] for the full set.

- `#![deny(unsafe_code)]` in all crate roots (no
  exceptions; unsafe belongs upstream)
- Clippy runs with `-D warnings` (zero tolerance)
- Errors via `thiserror`
- Logging via `tracing`
- Use workspace dependencies (`[workspace.dependencies]`)
  to keep versions consistent across crates
- Keep dependencies light. Avoid new dependencies
  when feasible
- Only add dependencies with well-established
  reputation
- `cargo audit` and `cargo deny check` enforce supply
  chain safety (see [getting-started.md])

[Cargo.toml]:../../Cargo.toml
[getting-started.md]:./getting-started.md

### Lint Suppression Policy

By default, do _not_ suppress lints. Use your best
judgement if the situation really calls for it.

Use `#[expect(...)]` instead of `#[allow(...)]`. The
`allow_attributes` lint enforces this mechanically.
Every suppression must include a `reason`:

```rust
// Good:
#[expect(
    clippy::too_many_lines,
    reason = "pipeline setup is inherently sequential"
)]
fn build_pipeline() { /* ... */ }

// Bad — denied by allow_attributes:
#[allow(clippy::too_many_lines)]
fn build_pipeline() { /* ... */ }
```

`#[expect]` is self-cleaning: if the suppressed lint
stops firing (because the code changed), the compiler
warns that the expectation is unfulfilled. This
prevents stale suppressions from accumulating.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [praxis-proxy/praxis](https://github.com/praxis-proxy/praxis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
