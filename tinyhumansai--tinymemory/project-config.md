---
trigger: always_on
description: This file is the single source of truth for how humans and coding agents work
---

# Repository Guidelines

This file is the single source of truth for how humans and coding agents work
in this repository. `CLAUDE.md` is a symlink to this file, so every agent reads
the same instructions.

Adapt the project-specific parts as the crate changes, and delete guidance that
no longer applies rather than leaving it to rot.

## Project Structure

This is a Cargo **workspace** with a virtual root: there is no root package,
and every crate lives in its own directory under `crates/`, named for the
package it holds. There are exactly three, one per part of the memory layer:

- `crates/tinymemory-api` — the **core contract**: `MemoryEngine`, items,
  metadata, namespaces, errors, and (feature `conformance`) the suite every
  engine must pass. No I/O.
- `crates/tinymemory-tools` — the **agent tool spec**: `MemoryTools` over any
  engine, holistic recall and the `context.md` compiler, and the agent memory
  lifecycle (`AgentMemory`, `Brain`, `MemoryLayout`, background jobs).
- `crates/tinymemory-integrations` — the **integrations**: the CortexDB engine
  and its registry, documents, sources, safety and the legacy v1 import, each
  a module behind a feature.

Shared package metadata, the version and the lint table live in the root
`Cargo.toml` (`[workspace.package]`, `[workspace.lints]`). The accepted
behaviour is [`docs/specs/memory-v2.md`](docs/specs/memory-v2.md); how it is
built is in [`docs/architecture/`](docs/architecture/README.md).

See [`README.md`](README.md) for the full layout and the feature table.

```text
crates/<package>/
├── Cargo.toml          # one package; `[lints] workspace = true`
├── README.md           # required of complex crates: design, surface, caveats
└── src/
    ├── lib.rs          # crate docs + the entire public re-export surface
    ├── error/mod.rs    # crate-wide `Error` and `Result<T>`
    └── <feature>/      # one directory per feature area
        ├── mod.rs      # module docs, wiring, smallest useful public API
        ├── types.rs    # substantial type definitions
        └── mod_tests.rs # module-local unit tests
crates/<package>/tests/     # integration tests against the public API only
crates/<package>/examples/  # runnable, compiled-in-CI usage examples
docs/
├── specs/              # behavior and architecture specifications
├── plans/              # test-first implementation plans
└── adr/                # immutable architecture decision records
```

Do not add a fourth crate. New contract surface goes in `tinymemory-api`, a new
agent-facing tool in `tinymemory-tools`, and anything that talks to the outside
world (a new engine, reader or converter) becomes a module of
`tinymemory-integrations` behind a feature named after it, with its
dependencies optional and enabled only by that feature.

Each feature area belongs in a focused module directory under the crate's
`src/`. A module root explains the module, wires its pieces together, and
exposes the smallest useful API. Move substantial type definitions into
`types.rs` and put module-local unit tests in a sibling `<module>_tests.rs`, wired from
the bottom of the module root with:

```rust
#[cfg(test)]
#[path = "mod_tests.rs"]
mod tests;
```

Do not write inline `mod tests` blocks in implementation files, do not name a test
file `test.rs`, `tests.rs` or `<module>_test.rs`, and do not
let a general-purpose `utils.rs` or `helpers.rs` grow — those are a symptom of a
missing module. Prefer many small modules that each do one thing well over few
broad ones.

Keep public exports centralized in each crate's `src/lib.rs` so downstream
users have one predictable surface. Put shared error variants in
`src/error/mod.rs` and return the crate-wide `Result<T>` from fallible public
APIs.

## Build And Test

Run every command from the repository root. These four are the contract; CI
runs exactly them, so a green local run should mean a green CI run.

```sh
cargo fmt --all -- --check
cargo clippy --all-targets --all-features -- -D warnings
cargo build --all-targets --all-features
cargo test --all-features
```

Supporting commands:

- `cargo fmt --all` — format before committing.
- `cargo test <filter>` — run a focused subset while iterating.
- `cargo run -p tinymemory-integrations --example basic` — run the bundled example. The
  `-p` is required: the workspace root is virtual, so cargo cannot infer which
  package an example belongs to.
- `cargo doc --no-deps --all-features` — build the rustdoc CI also builds with
  `RUSTDOCFLAGS="-D warnings"`.
- `cargo test --doc` — run doctests alone when editing documentation examples.

Never skip, ignore, or delete a failing test to make a command pass. Fix the
root cause, or stop and report the blocker.

## Coding Style

Use standard `rustfmt` output and Rust 2024 idioms. Do not hand-format around
`rustfmt`, and do not add `#[rustfmt::skip]` without a comment explaining why.

- `snake_case` for modules, files, functions, methods, fields, and locals.
- `PascalCase` for types, traits, and enum variants; `SCREAMING_SNAKE_CASE` for
  constants and statics.
- Name things for what they are, not for their layer: `RetryPolicy`, not
  `RetryHelper`.
- Prefer small, typed APIs over stringly-typed ones. Accept `&str` and generic

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tinyhumansai/tinymemory](https://github.com/tinyhumansai/tinymemory) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
