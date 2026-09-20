---
trigger: always_on
description: Engineering guidance for agents working in this repository. Read this alongside
---

# AGENTS.md

Engineering guidance for agents working in this repository. Read this alongside
`README.md`, the relevant crate documentation, and the code before making changes.

## Priorities

- Keep changes small, explicit, and focused on the root cause.
- Preserve user and other agents' changes. Never revert unrelated edits.
- Prefer existing standard traits and generics over unnecessarily concrete APIs.
- Protect protocol compatibility, bounded resource use, and UI responsiveness.
- Add regression tests for changed behavior and report what was actually verified.

## Architecture

Herdr GPUI is a native client of an existing local Herdr daemon, not a terminal
emulator, server, or TUI wrapper. Herdr owns terminal processes and session state.
Closing or detaching the GUI must leave the daemon and its terminals running.

| Location | Responsibility |
| --- | --- |
| `crates/herdr-protocol` | Generation-1 wire types, framing, validation, and atomic surface patches |
| `crates/herdr-client` | Discovery, socket worker, session transitions, ordered commands, and client-local activity projection |
| `crates/herdr-gpui` | Native UI, connection bridge, presentation state, semantic input, geometry, and painting |
| `crates/test-support/sandbox.rs` | Shared isolated process setup for opt-in integration tests; not a production crate |

- Keep dependency direction from UI to client to protocol. Protocol/client code must not depend on GPUI.
- Keep session transitions separate from socket scheduling, and connection ownership separate from window rendering.
- Reuse `ConnectionBridge`, domain targets, geometry helpers, and the test sandbox rather than duplicating their policies.
- Split modules by responsibility, not arbitrary line counts. Prefer wiring and exports in entry points as code grows; do not perform unrelated file reshuffles.
- Keep APIs narrow. Use private items by default and `pub(super)` or `pub(crate)` only where needed; do not expose every field merely to ease extraction.
- Define shared types once in the lowest appropriate layer and re-export them. Do not create mirror enums or convert between them through strings.
- Do not add a crate, runtime, service container, or speculative transport abstraction without a concrete need. Sibling checkouts are references, not build dependencies.

## Rust Conventions

- Require only the capabilities an operation uses. Prefer existing traits such as `Read`, `Write`, `IntoIterator`, `AsRef<Path>`, and `AsRef<OsStr>` over specific streams, collections, or owned paths.
- Use slices, `&str`, and `&Path` when borrowing is sufficient. Keep owned data where it must cross a thread boundary or outlive its caller.
- Prefer `impl Trait` or named generics for static dispatch. Use `dyn Trait` for actual runtime heterogeneity/type erasure, not by default.
- Use `FnOnce`, `FnMut`, or `Fn` for injectable behavior. Do not invent a callback or clock trait when a closure or explicit time value suffices.
- Do not introduce custom traits merely to wrap one struct. A new trait needs a concrete behavioral contract and a real implementation/substitution need that existing traits cannot express.
- Use enums for closed alternatives such as navigation targets, pane/popup input destinations, connection phases, and launch modes. Avoid string dispatch and boolean-plus-ID pairs internally.
- Parse strings at boundaries, then match on domain types. Prefer `From`, `TryFrom`, `Into`, and `TryInto` for conversions.
- Prefer typed request/result structures when shapes are known. Preserve genuinely open-ended protocol envelopes rather than forcing a speculative schema.
- Derive standard traits such as `Default`, `PartialEq`, and `Eq` where their semantics are valid. Compare complete values when deduplicating operations.
- Propagate errors with `?`; retain actionable categories and source/context until the display boundary. Implement meaningful `Display` and `Error`, not debug-only user messages. Do not add error-framework dependencies for trivial wrapping.
- No `unwrap()` or `expect()` in production. Test-only allowances must be scoped to test code. Keep unsafe exceptions narrow and document their safety conditions.
- Prefer guard clauses and readable iterators. Avoid clones, allocations, helper layers, and generic parameters that provide no benefit.
- Comments should explain invariants, ownership, or non-obvious decisions, not narrate assignments.

## GPUI Rules

- Treat app, window, and entity contexts as UI-thread execution. A foreground `spawn` future does not make blocking work safe.
- Never perform blocking socket/disk I/O, process waits, sleeps, or expensive background computation on the UI thread. Use the existing worker threads or GPUI background executor, then apply results through the appropriate context update.
- Keep render and input paths thin. Render from prepared state and bounded caches; do not query the daemon, scan files, or launch processes while rendering.
- Preserve continuously drained ordered events and the coalesced latest-state mailbox. Do not replace them with an unbounded UI event queue.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [penso/herdr-gpui](https://github.com/penso/herdr-gpui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-20 -->
