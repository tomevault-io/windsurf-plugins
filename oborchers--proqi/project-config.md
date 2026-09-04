---
trigger: always_on
description: These rules apply to all work in this repository. Read `context/PRODUCT.md`
---

# Proqi engineering contract

These rules apply to all work in this repository. Read `context/PRODUCT.md`
completely before changing visible behavior. Read `context/ARCHITECTURE.md`
completely before changing implementation boundaries or durable contracts.

`AGENTS.md` is the canonical instruction file at every scope. Each one must have
a sibling relative `CLAUDE.md` symlink that points to it. Never duplicate or
independently edit instruction content in `CLAUDE.md`.

## Verification

- `cargo xtask check` is the canonical local quality gate.
- Run focused tests while developing, then the complete gate before committing.
- Behavior changes require tests that prove the behavior and important failure
  paths. Bug fixes require a regression test where practical.
- Use `cargo xtask audit` and `cargo xtask package` at milestone and release
  gates. Coverage is a floor, not evidence that critical invariants are correct.
- Never weaken a gate, threshold, or test merely to make a change pass.
- Never auto-accept snapshots or golden files. Review their diffs explicitly.
- Before `1.0`, an intentional breaking CLI or JSON change must update the
  current-contract fixtures and the prepared GitHub Release notes.
- Every visible TUI change updates the representative Insta snapshots in the
  same commit. Pending `.snap.new` files fail the canonical quality gate.

## Architecture

Dependencies point inward:

```text
domain <- ports <- application <- adapters and UI composition
```

- Domain owns entities, typed identifiers, value types, and invariants. It does
  not import application, ports, adapters, terminal, SQL, process, filesystem,
  environment, or clipboard implementations.
- Ports describe terminal-independent capabilities in domain terms.
- Application depends on domain values and ports, never concrete adapters or UI.
- Adapters translate external systems into ports. UI translates input into
  application actions and renders application state.
- SQLite belongs below `src/adapters/sqlite`, Crossterm below the terminal
  adapter, Ratatui below UI, and child process execution below a process adapter.
- Time, IDs, paths, environment, filesystem, clipboard, and child processes use
  injected ports where behavior must be deterministic or platform independent.
- Keep one canonical import path for each public type. Implementation modules
  stay private, and adapter internals prefer `pub(crate)` or `pub(super)`.
- Constructors enforce invariants. Keep fields private when direct mutation
  could produce invalid state.

## Single-source ownership and module structure

- When the same semantic rule, label, geometry calculation, protocol token, or
  policy has three consumers, move it to one named owner in the innermost
  appropriate layer. Two consumers should share it when drift would produce an
  incorrect or unsafe result. Do not create a generic utility module without a
  clear domain owner.
- Rendering, measurement, hit testing, help, and command dispatch derive from
  the same semantic definitions. Do not keep parallel constants or reconstruct
  one consumer's behavior from another consumer's formatted output.
- Prefer typed identifiers, enums, value objects, and exhaustive matching for
  closed sets. Keep external compatibility strings at translation boundaries;
  do not use strings as internal dispatch keys when the variants are known.
- A behavior-preserving refactor preserves public Rust paths, CLI and JSON
  spelling, durable encodings, error classification, and snapshot content unless
  the task explicitly authorizes a contract change.
- Split modules by responsibility and ownership, not merely to satisfy a line or
  complexity limit. Moving unrelated functions into a vaguely named file or
  compacting readable code is not an acceptable lint fix.
- Keep behavior-owned tests beside the implementation, using an adjacent
  `tests.rs` or `tests/` module when inline tests obscure production code. Move
  cross-layer contracts, process tests, and PTY scenarios to top-level `tests/`.
  The 500-line ceiling applies equally to production and test source.
- Before introducing a new abstraction, find the current canonical owner and
  extend it when the responsibility matches. Remove superseded paths in the same
  change so two sources of truth cannot remain live.

## Rust guardrails

- Unsafe Rust is forbidden unless an explicit architecture decision documents
  and reviews an unavoidable need.
- Production code does not use `unwrap`, `expect`, `panic!`, `unreachable!`,
  `todo!`, or `unimplemented!`. Return typed errors. A proven invariant may use
  a narrow
  `#[expect(..., reason = "...")]` instead of a broad allow.
- Every first-party implementation source file is at most 500 physical lines.
  Public documentation and prose such as `README.md` and `context/*.md` are
  exempt; preserve the detail those contracts need.
- Rust functions are limited by the checked-in Clippy cognitive-complexity,
  function-length, and nesting thresholds.
- Keep tests deterministic. Inject clocks, identifiers, paths, and process
  execution. Do not depend on test order, wall-clock timing, or user state.
- Preserve complete prefixed UUIDv7 values at every boundary. Do not accept

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [oborchers/proqi](https://github.com/oborchers/proqi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-04 -->
