---
trigger: always_on
description: Conventions for humans and AI agents contributing to this repo.
---

# Working on Once

Conventions for humans and AI agents contributing to this repo.

## GitHub

- Use Conventional Commit style for pull request titles, such as
  `fix(docs): update deployment route`.
- Do not prefix pull request titles with `[codex]`.

## Module Layout

**Avoid monolith Rust files.** When a `lib.rs` or `main.rs` grows past
roughly 200 lines or starts mixing unrelated concerns, split it into a
module per responsibility. The top-level file should read as a table of
contents: re-exports, `mod` declarations, and dispatch.

Tests live next to the code they exercise: `#[cfg(test)] mod tests { ... }`
inside each module file. Cross-module integration tests go under
`crates/<crate>/tests/`.

## Manifest Files

- Per-package manifests are named `once.toml`.
- The `.once/` directory at the workspace root is cache and runtime
  state, not a manifest. It is gitignored.
- Root `once.toml` configures workspace-level settings such as cache
  providers. Package `once.toml` files may grow build graph declarations
  as Once expands beyond script-only workflows.
- Scripts declare their execution metadata with `# once` headers in the
  script file.

## Scope

Once starts with cacheable and remotely executable scripts plus the
runtime API, and is expanding toward build-system capabilities. Scripts
remain the migration ramp into the Once build graph. Build graph work
should follow RFCs and keep the agent-facing graph model typed,
queryable, and structurally editable.

Keep the current CLI surface centered on:

- `once exec` for literal commands and annotated script files
- `once cache` for CAS and action-cache primitives
- `once runtime` for JSON-RPC runtime sessions
- `once auth` and `once toolchain` for supporting infrastructure

New build graph CLI, target kind, and query surfaces should be introduced
deliberately and documented in the relevant RFC or implementation plan.

Generic surfaces must stay ecosystem-neutral. CLI commands, Rust APIs,
MCP tools, and shared graph/query records should not hardcode examples,
field names, or behavior around one toolchain such as Apple, Cargo,
npm, SwiftPM, or crates.io. Put ecosystem-specific behavior behind a
resolver or Starlark target kind implementation, or behind a dedicated
toolchain guide/reference page so future ecosystems can plug into the
same shape instead of requiring parallel CLI or MCP surfaces.

## Graph Modules

Once exposes a doc-less surface for coding agents: an agent should be
able to discover what target kinds exist, pull a runnable starter, validate a
draft, and commit an edit using MCP tool calls alone, without reading
prose docs. When adding support for a new toolchain (Android, JVM,
Rust, etc.), mirror the shape already established for the Apple target kinds
rather than inventing a parallel surface.

Rust code must stay toolchain-agnostic. Do not add Rust branches that
recognize Apple, Android, JVM, Rust, or any other build system by name.
Build system behavior belongs in target kinds. The Rust side should provide
generic primitives, typed graph plumbing, validation surfaces, and
execution policy that target kinds can compose to express their needs.

Starlark target kind contract changes must update the public Starlark modules
reference in the same change. This includes new globals, changed `ctx`
fields, action declaration semantics, provider expectations, schema
helpers, loading behavior, or project module authoring guidance. Shared
Starlark helpers should live in the common prelude instead of being
copied into each toolchain file. The Starlark prelude index owns the
built-in module source order, so adding or removing a target kind family should
not require Rust executor changes.

Every new toolchain target kind should preserve these invariants:

- The target kind is discoverable through `once_list_target_kinds` and its full
  contract is fetchable through `once_query_schema`.
- The target kind ships at least one runnable starter example as a real
  directory owned by the module package (manifest plus sources). The
  exported Starlark target kind symbol declares it through
  `example(slug, name = ..., use_when = ..., path = ...)`; `path`
  defaults to `examples/<slug>`, and inline TOML strings are not
  allowed. Example paths are relative to the module package and must stay
  inside it so target kinds can be published through a registry as a portable
  unit.
- Every example loads under the examples integration test
  (`crates/once-frontend/tests/examples.rs`) without emitting any
  diagnostics. If the target kind has an `impl`, the example must build.
- User-visible failures surface through the structured `Diagnostic`
  shape (`code`, `target`, `attribute`, `repairs`). Validation lives
  in `target_validator`; the editor in `manifest_editor` reuses the
  same shape so retries are single-shot for the agent.
- Every MCP tool has a matching `once query` or `once edit` CLI
  subcommand so a human can reproduce what an agent does from the
  terminal.

The built-in Apple target kinds and their portable starter examples are the
reference implementation. Treat their module-owned examples, schema
metadata, validation, and MCP/CLI discovery shape as the template when
wiring a new toolchain.

## SDK API And Docs


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tuist/once](https://github.com/tuist/once) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-27 -->
