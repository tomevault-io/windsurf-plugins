---
trigger: always_on
description: Thank you for contributing to Coffer.
---

Contributing guide
==================

Thank you for contributing to Coffer.

This document is also the operating guide for coding agents working in this
repository. *AGENTS.md* and *CLAUDE.md* are symbolic links to this file.

Before changing code, read:

1.  [*README.md*](./README.md), for the purpose and security posture of the
    project;
2.  [*ROADMAP.md*](./ROADMAP.md), for current scope and milestone ordering; and
3.  this document in full.

If you use an AI coding agent, read and follow [*AI\_POLICY.md*](./AI_POLICY.md)
first.

When instructions conflict, favor credential safety, data integrity, license
compliance, and the narrower interpretation of the requested change.


Development environment
-----------------------

Coffer uses [mise] as the single entry point for project tooling.

Do not add instructions that require contributors to manage the Coffer Rust
toolchain with `rustup`, and do not assume a globally selected Rust version.

Install the project's pinned tools with:

~~~~ sh
mise install
~~~~

Inspect available tasks with:

~~~~ sh
mise tasks
~~~~

The Rust toolchain configured in *mise.toml* must include:

 -  rust-analyzer;
 -  `rustfmt`; and
 -  Clippy.

Auxiliary tools, including [Hongdown], [cargo-deny], [actionlint], and
[zizmor], must likewise be managed through mise where practical.

Linux builds also require `pkg-config` and OpenSSL development headers because
the Secret Service adapter selects oo7's `openssl_crypto` backend to avoid an
unwiped plaintext buffer in oo7 0.6.0's `native_crypto` implementation.

Avoid introducing ad hoc setup commands when the operation should instead be a
reproducible mise task.

[mise]: https://mise.jdx.dev/
[Hongdown]: https://github.com/dahlia/hongdown
[cargo-deny]: https://github.com/EmbarkStudios/cargo-deny
[actionlint]: https://github.com/rhysd/actionlint
[zizmor]: https://github.com/zizmorcore/zizmor


Canonical development tasks
---------------------------

The repository should expose the following task interface through *mise.toml*.

### `mise run fmt`

Format all project-owned source and documentation.

At minimum this includes:

~~~~ sh
cargo fmt --all
hongdown --write
mise fmt
~~~~

If new languages or generated sources are added, extend the mise task rather
than requiring contributors to remember an additional formatter.

### `mise run fmt-check`

Verify formatting without modifying the working tree.

At minimum this includes:

~~~~ sh
cargo fmt --all --check
hongdown --check
mise fmt --check
~~~~

### `mise run check`

Run fast static verification suitable for frequent use and pre-commit checks.

This should include:

 -  Clippy for the entire workspace and all relevant targets, which runs the
    full compiler front end and therefore subsumes `cargo check`;
 -  `rustfmt` verification;
 -  Hongdown verification;
 -  mise configuration formatting verification;
 -  GitHub Actions workflow linting; and
 -  checks for other project-owned languages when they are introduced.

Checks that need the network, such as the dependency audit, do not belong here.

### `mise run build`

Build every workspace target, including tests, examples, and benchmarks,
without running them.

### `mise run test`

Run the normal test suite for the complete workspace.

Tests that access a live Apple Account or perform security-sensitive network
operations must not be part of this default task.

### `mise run doc`

Build Rust API documentation with warnings denied.

Public Rust APIs are expected to remain useful and reviewable through rustdoc.

### `mise run deny`

Audit the dependency graph with cargo-deny against *deny.toml*: security
advisories, license policy, banned crates, and permitted sources.

This task fetches the RustSec advisory database and therefore needs network
access.

### `mise run ci`

Run the complete local verification gate expected to match continuous
integration. It runs `check`, `build`, `test`, `doc`, and `deny`; continuous
integration runs exactly this task and nothing else.

A change is not complete until `mise run ci` succeeds.

Every Cargo invocation in the gate that resolves dependencies passes
`--locked`. When a change to a *Cargo.toml* requires a lockfile update, update
*Cargo.lock* deliberately with `cargo update` or `cargo add` and review the
result before running the gate.

If one of these tasks is absent while bootstrapping the repository, add the task
instead of bypassing the intended interface.


Rust quality policy
-------------------

Coffer has a zero-warning policy.

The workspace should configure:

~~~~ toml
[workspace.lints.rust]
warnings = "deny"

[workspace.lints.clippy]
all = "deny"
~~~~

The workspace additionally denies missing documentation on public items,
denies `unsafe` code unless it is allowed in a narrow scope, and requires every
`unsafe` block to carry a `SAFETY:` comment through Clippy's
`undocumented_unsafe_blocks` lint. The `coffer-protocol` crate goes further and
forbids `unsafe` at the crate level; if FFI ever has to live there, relax that
attribute to `deny` deliberately before adding a scoped allowance.

Workspace crates must inherit the workspace lint configuration with
`[lints] workspace = true`. The Clippy task also passes `-D warnings` on the

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dahlia/coffer](https://github.com/dahlia/coffer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-04 -->
