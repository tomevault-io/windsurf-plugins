---
trigger: always_on
description: This file defines repository-specific rules for AI coding agents working on
---

# Robonix Agent Guidelines

This file defines repository-specific rules for AI coding agents working on
Robonix. Read once at session start; obey throughout.

## Concept and naming stability

- `docs/src/developer-guide.md` is the source of truth for concepts and
  terminology. When code and the dev guide disagree, fix the code, not
  the guide.
- The formal concepts are **capability** (the interface) + **contract**
  (its shape) + three provider kinds **primitive / service / skill**.
  Do not introduce additional umbrella terms in user-facing prose
  (dev guide, README, quickstart, CLI output, error messages, commit
  messages). The internal Rust type `CapabilityProvider` / Python
  `_ProviderBase` / fields `provider_id`, `provider_kind` are
  implementation labels and stay as-is.
- No new concept / rename / RPC / state name without explicit user
  confirmation in the conversation. If you find yourself about to
  introduce a noun or verb that is not already in the dev guide, **stop
  and ask** before writing code.

## Project Structure & Module Organization

Robonix is an **embodied AI operating system**: a monorepo with Rust system
components, Python packages, capability contracts, IDL libraries, examples, and
documentation. The Rust workspace lives at the repository root in `Cargo.toml`;
current Rust members are `system/{atlas,executor,liaison,pilot}` and
`tools/{rbnx,codegen}`. The twelve system component directories live under
`system/`; several are documentation/planning components today, while
`system/scene` is a Python package managed by uv. Python workspace packages are
listed in the root `pyproject.toml`: shared libraries in `pylib/*`, reference
services in `services/*`, and `system/scene`. Capability contracts are TOML
files under `capabilities/{primitive,service,system}/`; reusable message and
service IDL libraries live under `capabilities/lib/`. End-to-end and Webots
examples live in `examples/`, Docker packaging in `docker/`, repo scripts in
`scripts/`, image assets in
`images/`, and the mdBook manual in `docs/src/`.

### Crate README maintenance

When you add or change functionality in a Rust workspace member under
`system/*` or `tools/*`, update that member's `README.md` in the same
contribution. If the member has no `README.md`, add one. Each README should be
concise and informative: what the package is, what it is for, and how to use
it. For CLIs and similar tools, document subcommands, flags, and important
environment variables.

## Build, Test, and Development Commands

Run Rust commands from the repository root unless noted:

- `make build`: build all Rust workspace crates in debug mode.
- `make release`: build all Rust crates with `--release`.
- `make install`: install `rbnx`, Atlas, Pilot, Executor, Liaison, and Codegen
  to `~/.cargo/bin`.
- `make check`: run `cargo fmt --all -- --check` and clippy with warnings
  denied.
- `cargo test --workspace --all-targets`: run unit and integration tests.
- `uv sync`: resolve Python workspace dependencies from the repo root; service
  `scripts/build.sh` files usually perform package-local setup.
- `cd docs && mdbook build`: build the documentation book when mdBook and
  preprocessors are installed.

## Coding Style & Naming Conventions

Rust uses edition 2024 and standard `rustfmt`; keep clippy clean with
`-D warnings`. Use `snake_case` for Rust modules/functions, `CamelCase` for
types, and crate names matching `robonix-*`. Python targets 3.10+, uses
4-space indentation, and keeps package modules in `snake_case`. Capability
files follow `<interface>.v1.toml` naming, grouped by domain, for example
`capabilities/primitive/camera/rgb.v1.toml`.

If one function/method body exceeds five lines, treat documentation as mandatory: The comment block must state behavior at minimum; add side effects, or constraints when they are not obvious from the signature.

## Testing Guidelines

Place Rust integration tests in each crate's `tests/` directory and keep unit
tests near the code behind `#[cfg(test)]`. CI currently gates Rust format,
clippy, build, and `cargo test --workspace --all-targets`. For Python services,
add package-local smoke tests or scripts when changing runtime behavior, and
document required environment variables in the service README.

Do not add production-code solely to make tests pass; if tests require
production changes, those changes must improve real behavior, design clarity,
diagnostics, or maintainability outside the test.

## Editing discipline

- Never run `sed` (or equivalent multi-file replace) across more than one file
  at a time during a rename. Read each file before editing. Pattern variables
  that look identical at the string level can have different types; blanket
  replace breaks tuple destructures and loop variables.
- Do not hand-edit generated documentation under `docs/src/reference/`
  (including `idl.md` and `contracts.md`). Update the source contracts / IDL /
  codegen inputs, then regenerate docs with the project tooling when generated
  reference pages need to change.
- `cargo fmt --all && cargo clippy --workspace --all-targets -- -D warnings`
  must be clean before any commit that touches Rust.
- Don't commit without an end-to-end run when the change crosses process

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [syswonder/robonix](https://github.com/syswonder/robonix) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
