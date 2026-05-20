---
trigger: always_on
description: You are working on **WebUI**, a high-performance server-side rendering framework written in Rust that operates without JavaScript runtimes. It separates static and dynamic content at build time into a binary protocol that enables fast rendering in any host language.
---

# WebUI - Copilot Instructions

You are working on **WebUI**, a high-performance server-side rendering framework written in Rust that operates without JavaScript runtimes. It separates static and dynamic content at build time into a binary protocol that enables fast rendering in any host language.

Read and internalize these instructions at the start of every session. They are non-negotiable.

---

## Context you must load first

Before suggesting or applying **any** change, read these files - they are the ground truth:

1. **`DESIGN.md`** - The living technical specification. Architecture, protocol schema, module contracts, and behavioral rules all live here. Treat every constraint in it as mandatory unless the user explicitly asks to change one.
2. **`Cargo.toml`** (workspace root) - Workspace members, dependency versions, and release profile.
3. **`clippy.toml`** - Lint policy (bans `unwrap`/`expect`, caps cognitive complexity at 20, limits function arguments to 5).
4. **`deny.toml`** - Allowed licenses and advisory ignore-list.
5. The specific crate(s) under `crates/` that are relevant to the task at hand.

---

## The one rule that gates everything

Before creating **any** commit, run:

```bash
cargo xtask check
```

This executes, in order: `license-headers → fmt → clippy → deny → test → build`. Work is **not done** until this passes cleanly. If it fails, fix every reported issue before proceeding. No exceptions.

---

## Performance is the top priority

Every decision - API design, data structure choice, algorithm, error path - must be evaluated through a performance lens first. WebUI's value proposition is speed; nothing else matters if it is slow.

### Hard constraints

| Rule | Rationale |
|------|-----------|
| **No recursion** in core algorithms | Use iterative loops with an explicit stack. Recursion blows the call stack on deep templates and defeats branch prediction. |
| **No regular expressions** in core logic | Deterministic scanners are faster and more predictable. |
| **Minimal runtime computation** | Move work to build time (the `webui build` CLI step) whenever possible. |
| **Protobuf binary serialization** via `prost` | Zero-copy decoding; JSON is for `webui inspect` debugging only. |
| **Buffer consolidation** | Reuse buffers, pre-allocate, avoid unnecessary allocations. |

### Allocation discipline

- `Vec::with_capacity` / `String::with_capacity` when size is known or estimable.
- `push_str` / `write!` into existing buffers - never `format!` in hot paths.
- No unnecessary `.clone()` - pass `&str`, `&[T]`, or slices. Use `Cow<'_, str>` when a value is sometimes borrowed, sometimes owned.
- Prefer explicit state machines and stack-based traversal over recursive AST walking.
- Never clone large state trees for read-only lookups - use resolver closures or borrowed references.
- Never clone a `HashMap` to save/restore scope - save/restore only the overwritten key.
- Never call `.to_string()` on a `Cow` - write it directly to avoid defeating zero-copy.
- Iterate `split()` directly - never `collect::<Vec<_>>()` when sequential access suffices.
- For performance patterns (speed and memory), see: `skills/perf/SKILL.md`.

### Measurement

- Identify hot paths first: parsing, expression evaluation, handler rendering, protocol serialization, state lookups.
- Measure **before** changing anything: `cargo bench -p <crate>` in `--release` mode.
- After the change, re-measure and report the delta (qualitatively if exact numbers aren't available).
- The smallest safe change that improves CPU time, allocation count, or cache locality wins. Do not over-engineer.

---

## Rust architecture standards

### Error handling
- Library crates (`webui-parser`, `webui-handler`, `webui-expressions`, `webui-state`, `webui-protocol`, `webui-ffi`) use **custom error enums** via `thiserror`.
- Binary crates (`webui-cli`, `xtask`) may use `anyhow`.
- **No `unwrap()` or `expect()`** in library code - `clippy.toml` enforces this.
- Errors must be **actionable**: tell the caller what went wrong *and* what they can do about it.

### Public API surface
- Expose the minimum necessary. Use `pub(crate)` for internal helpers.
- New public functions, structs, traits, and error variants must be documented with `///` doc-comments.
- Use `#[must_use]` on functions whose return value should not be silently discarded.

### Code organization
- One concern per module. When a file approaches ~400 lines, split it.
- Types are `PascalCase`, functions are `snake_case`, constants are `SCREAMING_SNAKE_CASE`.
- `cargo fmt --all` is the sole formatting authority - never override or disable it.

### License header

Every source file (`.rs`, `.ts`, `.js`, `.cs`, `.h`, `.proto`) **must** start with:

```
// Copyright (c) Microsoft Corporation.
// Licensed under the MIT license.
```

followed by a blank line before any code. `cargo xtask check` enforces this. Run `cargo xtask license-headers --fix` to add missing headers automatically.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [microsoft/webui](https://github.com/microsoft/webui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
