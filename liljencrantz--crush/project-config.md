---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Crush is a command line shell that is also a modern programming language — closures,
lexical scoping, a type system, and a structured data pipeline (like a shell where
pipes carry typed tables/rows instead of bytes), written in Rust.

## Build / test / run

```
cargo build --release && cargo install --path .   # build + install to ~/.cargo/bin
cargo build                                        # debug build (produces ./target/debug/crush, used by tests)
cargo run                                          # run interactively
cargo test                                         # run all tests (see below)
```

OS deps must be installed before building (protobuf, openssl/libssl-dev, and on Linux
libdbus-1-dev + libsystemd-dev) — see README.md for the exact per-OS package lists.

### Tests

`cargo test` covers two very different things:

- Normal Rust unit tests scattered across the crates as usual (`#[test]` fns).
- **System/golden tests** under `tests/`: each `tests/<name>.crush` script is run against
  the *debug* binary (`./target/debug/crush`) and its stdout is compared line-by-line
  against `tests/<name>.crush.output`. These are auto-discovered by the `test_finder!()`
  macro (from the `test_finder` proc-macro crate) in `tests/system.rs` — dropping a new
  `foo.crush` + `foo.crush.output` pair into `tests/` is enough to add a new case, no
  Rust code needed. **Because these tests exec the debug binary, `cargo build` must be
  run (or be up to date) before `cargo test` picks up code changes** — `cargo test` alone
  does not rebuild the binary the system tests invoke.
- `test_grpc` builds and spawns the `grpc-service` sub-crate binary to exercise the gRPC
  client builtins.

To add/update a golden test: write the `.crush` script, run it manually through the
debug binary, and save the output as the matching `.crush.output` file.

Prefer this plain golden-test style over ad-hoc Rust assertions in `tests/system.rs`
(stderr/exit-code checks, etc.) — even for a bug whose only visible symptom seems to
need something other than a stdout diff (e.g. an async background thread's error). Two
techniques cover most such cases:

- If the only symptom is "the script aborts" (no message content to check): a
  top-level job's error aborts the rest of the script (see `execute.rs`'s `source()`),
  so performing the fragile operation and then `echo`ing a marker afterward turns "does
  this error?" into a plain stdout diff — the marker only appears if it didn't.
- If the error *message* itself matters: `try { <fragile op> } catch { |$e| $msg = $e }`
  captures the error as a plain string, which ordinary `assert`/`:like` calls can then
  check.

Only fall back to a Rust assertion in `tests/system.rs` when both are genuinely
impossible — and even then, prefer testing a structural property (e.g. "stderr is
non-empty") over coupling to exact error-message wording.

When a bug is found while working on something else, write its `.crush`/`.crush.output`
repro pair immediately, even before it's fixed — a known-failing test documents the bug
precisely and won't get lost or need rediscovering later.

## Crush language notes for script authors

A few non-obvious behaviors worth knowing before writing `.crush` test scripts or
tooling (e.g. `generate_docs.crush`) — the full semantics belong in
`docs/language_reference.md`, this is just the pitfalls that have cost real time:

- `:like`/`:not_like` do exact string match by design, not substring or wildcard
  matching — wildcarding is a property of a `glob` *value* passed as the pattern, not
  something the method name implies.
- Every codec's `:to` command (`json:to`, `yaml:to`, `pup:to`, ...) intentionally
  produces a `binary_stream`, not a plain string — this is deliberate ("bytes ready to
  write to disk or pipe onward"), not a quirk of any one codec. Use
  `convert $string $(...)` when a plain string is actually needed.
- String concatenation is not `+` (that's numeric/duration addition) — use `:format`.
- A regex literal's outer `^(...)` is Crush's own delimiter syntax, not itself capture
  group 1 — `^((.*)-(.*))` numbers its groups 1 and 2, not 2 and 3.

## Workspace layout

Cargo workspace with the main `crush` crate plus four local crates:

- `signature/` — proc-macro crate providing `#[signature(...)]`, the attribute used to
  declare every builtin command's argument struct (see below). This is the main piece
  of "framework" code in the project. Its generated code references `crate::lang::*`
  paths that only resolve inside the `crush` binary crate (which has no `[lib]` target,
  so `signature` can't depend on it), so a real macro expansion can only be compiled as
  part of `crush` itself — a `trybuild`-style compile-fail test in `signature/` can't
  isolate "rejected for the intended reason" from "failed because `crate::lang` doesn't
  exist here" without exact-stderr matching. `signature/src/lib.rs`'s own tests instead
  call `signature_real()` (the plain function the `#[proc_macro_attribute]` wrapper
  delegates to) directly and check `Result::is_err()`/`is_ok()` — see that file for the

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [liljencrantz/crush](https://github.com/liljencrantz/crush) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-27 -->
