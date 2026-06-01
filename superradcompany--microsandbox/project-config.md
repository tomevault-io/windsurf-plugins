---
trigger: always_on
description: These instructions are only for agents contributing changes to this repository.
---

# AGENTS.md

These instructions are only for agents contributing changes to this repository.
Do not apply them to other repositories or to general agent behavior outside this contribution workflow.

## Scope

- Follow these guidelines when editing code, docs, tests, examples, CI, or release files for microsandbox.
- Prefer repository conventions over generic agent habits. When unsure, inspect nearby files and match their style.
- Do not create branches, commit, push, tag, publish, or open pull requests unless the human explicitly asks.
- Check `git status --short --branch` before making changes. Do not overwrite or revert user work unless explicitly asked.

## Project Map

- `crates/microsandbox` is the public Rust SDK crate.
- `crates/cli` contains the `msb` CLI.
- `crates/runtime` contains VM runtime integration.
- `crates/filesystem`, `crates/image`, `crates/network`, `crates/db`, `crates/migration`, `crates/metrics`, `crates/protocol`, and `crates/utils` are shared internal crates.
- `crates/agentd` is the in-guest agent. It is excluded from the root Cargo workspace and is built separately.
- `sdk/python`, `sdk/node-ts`, and `sdk/go` contain the language SDKs and native bindings.
- `docs/` contains the documentation site. Keep docs in sync with user-facing behavior.
- `examples/` contains runnable examples. Add new example projects only when requested or clearly required by the contribution.
- `mcp/` and `skills/` are submodules related to agent integrations.
- `vendor/libkrunfw` is a submodule for the kernel firmware library.

Repository layout:

```text
.
|-- AGENTS.md
|-- Cargo.lock
|-- Cargo.toml
|-- DEVELOPMENT.md
|-- Dockerfile.agentd
|-- justfile
|-- msb-entitlements.plist
|-- assets/
|-- benchmarks/
|-- crates/
|   |-- agentd/
|   |-- cli/
|   |-- db/
|   |-- filesystem/
|   |-- image/
|   |-- metrics/
|   |-- microsandbox/
|   |-- migration/
|   |-- network/
|   |-- protocol/
|   |-- runtime/
|   |-- test-init/
|   |-- test-macros/
|   |-- test-utils/
|   `-- utils/
|-- docs/
|   |-- cli/
|   |-- getting-started/
|   |-- images/
|   |-- networking/
|   |-- recipes/
|   |-- sandboxes/
|   `-- sdk/
|-- examples/
|   |-- python/
|   |-- rust/
|   `-- typescript/
|-- mcp/
|   |-- bin/
|   |-- src/
|   `-- package.json
|-- packaging/
|   `-- docker/
|-- scripts/
|   `-- smoke/
|-- sdk/
|   |-- go/
|   |-- node-ts/
|   `-- python/
|-- skills/
|   `-- microsandbox/
`-- vendor/
    `-- libkrunfw/
```

## Design Principles

- microsandbox is still early software. Prefer the clean current design over compatibility layers, migration shims, deprecated paths, or version checks unless a maintainer asks for them.
- Keep changes narrowly scoped to the requested behavior. Avoid drive-by refactors, unrelated formatting, or dependency churn.
- Treat sandbox isolation, host filesystem access, networking, and secret handling as security-sensitive. Validate inputs at boundaries and avoid exposing host paths, credentials, or ambient privileges.
- For public APIs, keep the Rust SDK, CLI, Python SDK, Node SDK, Go SDK, docs, and examples consistent when they describe the same capability.
- Prefer explicit errors with useful context over silent fallbacks.

## Rust Layout And Style

- Most Rust crates use `lib/lib.rs` for library code and `bin/main.rs` for binaries. Keep using those paths for new crate entries unless the surrounding crate already does something different.
- When adding a new library or binary target, declare the path explicitly in `Cargo.toml`:

```toml
[lib]
path = "lib/lib.rs"

[[bin]]
name = "example"
path = "bin/main.rs"
```

- Keep crate roots and module roots thin. `lib.rs` and `mod.rs` should declare modules, crate attributes, and exports only. Put implementation in leaf modules such as `sandbox/config.rs`, `policy/types.rs`, or `commands/run.rs`.
- File order should be:
  1. Module docs and crate/file attributes, such as `//! ...` and `#![warn(missing_docs)]`.
  2. `use` imports.
  3. Sectioned items.
- Group imports by origin, separated by blank lines: standard library first, external crates second, then `crate::` and `super::` imports.
- Do not put `use` statements inside sections unless there is a narrow local reason, such as a test module import.
- Use the exact section delimiter shown below. Do not invent alternate Markdown-style, shorter, or decorative section headers.
- Include only sections that contain items. Do not add empty sections just to satisfy the full order.
- Organize Rust files with these section headers, in this order when applicable:

```rust
//--------------------------------------------------------------------------------------------------
// Constants
//--------------------------------------------------------------------------------------------------

//--------------------------------------------------------------------------------------------------
// Types
//--------------------------------------------------------------------------------------------------

//--------------------------------------------------------------------------------------------------
// Methods
//--------------------------------------------------------------------------------------------------


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [superradcompany/microsandbox](https://github.com/superradcompany/microsandbox) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->
