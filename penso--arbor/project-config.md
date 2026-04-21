---
trigger: always_on
description: This file defines how coding agents should behave in this repository.
---

# AGENTS.md

This file defines how coding agents should behave in this repository.

## Priorities

1. Keep code simple, explicit, and maintainable.
2. Fix root causes, avoid temporary band-aids.
3. Preserve user changes, never revert unrelated edits.

## Workflow

1. Read this file at task start.
2. Prefer `just` recipes for common tasks.
3. Before committing code, always run `just format` and `just lint` and fix any failures.
4. Before handoff, run relevant checks for touched code.

## UI Parity

- Arbor has two user-facing UI surfaces: `arbor-gui` and `arbor-web-ui`.
- When adding, changing, or removing a user-visible feature in one surface, check whether the other surface needs the same capability.
- Default to keeping both surfaces in parity. If parity is intentionally deferred, call that out clearly in the handoff and create follow-up work instead of silently shipping only one side.
- UI verification should cover both surfaces when the feature is meant to exist in both.

## GPUI Threading Rules

- Treat the GPUI app/window/entity context as the UI thread unless you have explicitly moved work off it.
- Be extremely careful not to block the UI thread with disk I/O, network I/O, daemon RPCs, SSH, git/process spawning, sleeps, waits, or CPU-heavy work. If it can stall a frame, assume it is forbidden on the UI thread.
- GPUI/Zed guidance matters here: `App::spawn`, `Context::spawn_in`, and `AsyncWindowContext::spawn` run futures that are polled on the main thread. Do not put blocking or CPU-intensive work directly inside those futures.
- Use `cx.background_spawn(...)` or a `BackgroundExecutor` for blocking or CPU-heavy work. Use the foreground task only to kick work off, await the result, and then hop back into `update(...)` to apply state changes.
- If you must adapt a synchronous blocking function, run it on a background thread/executor. Prefer GPUI background tasks, and use `smol::unblock` when you need to wrap a blocking call into a future. Do not introduce Tokio.
- Render paths must be pure state reads. Never do filesystem scans, config loads, git queries, daemon calls, SSH work, process spawning, or expensive recomputation from `render_*` methods.
- Event handlers and hot paths must stay thin. `on_action`, `listener`, websocket/message handlers, timers, auto-refresh loops, and key/mouse handlers should schedule background work and return quickly instead of doing the slow part inline.
- Cache derived data that is expensive to compute or load. If the UI needs it often, compute it in the background, store it in state, and render from the cached state.
- When reviewing GPUI code, ask two questions every time: `could this block?` and `could this run during render or a hot UI path?` If yes, move it off-thread.

## Commands

- Format: `just format`
- Format check: `just format-check`
- Lint: `just lint`
- Test: `just test`
- Run app: `just run`
- Run HTTP daemon: `just run-httpd`

## Rust Rules

- Do not use `unwrap()` or `expect()` in non-test code. In test modules, allow them with `#[allow(clippy::unwrap_used, clippy::expect_used)]`.
- Use `arbor_core::ResultExt` / `arbor_core::OptionExt` for `.context()` instead of ad-hoc `.map_err()`.
- Use `SessionId` and `WorkspaceId` newtypes (from `arbor_core::id`) instead of raw `String`.
- Use clear error handling with typed errors (`thiserror`/`anyhow` where appropriate).
- Keep modules focused and delete dead code instead of leaving it around.
- **Never shell out to external CLIs** (`gh`, `git` via `Command::new`, etc.) for GitHub API calls. Use Rust crates (`octocrab`, `reqwest`, etc.) instead. `std::process::Command` is if no rust crate exists.

## Code Organization

- Split large files by domain. Target ~800 lines per file max.
- Use `pub(crate)` for items shared within a crate. Apply to fields, methods, and free functions.
- Use `pub(crate) use module::*` re-exports to keep call sites clean after splitting.
- When extracting code to a new file, ensure all struct fields are `pub(crate)` (not private) so the parent module can access them.

## Feature Flags

- `arbor-gui` features: `ssh`, `mosh`, `mdns` (default: all enabled).
- `arbor-httpd` features: `mdns` (default: enabled).
- `arbor-core` features: `ssh`, `mosh` (propagated from GUI).
- Gate optional modules/functions with `#[cfg(feature = "...")]`.
- Use `dep:crate_name` syntax for optional dependency features.
- `mosh` implies `ssh` — features are hierarchical.

## Workspace Dependencies

All dependency versions live in the root `Cargo.toml` `[workspace.dependencies]`. Subcrate `Cargo.toml` files use `{ workspace = true }`. Never hardcode a version in a subcrate.

## Common Mistakes to Avoid

- **Missing `pub(crate)` on extracted items**: When moving structs, enums, or functions from `main.rs` to submodules, every field and method that was previously accessible needs `pub(crate)`.
- **Unused imports after extraction**: After moving code out of a file, clean up `use` statements in the source file — the compiler will flag these.
- **Duplicate definitions**: When splitting code, ensure a function/type exists in exactly one place. Check both the source and destination files.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [penso/arbor](https://github.com/penso/arbor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
