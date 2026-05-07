---
trigger: always_on
description: <!-- Actioneer-gtk: Copilot / AI agent instructions -->
---

<!-- Actioneer-gtk: Copilot / AI agent instructions -->
# Quick guide for automated coding agents

This repository is a native GTK4/libadwaita desktop client for GitHub Actions written in Rust. The notes below focus on the patterns and files an AI coding agent should know to make safe, useful changes quickly.

Important local docs
- The repository contains a curated, local copy of Libadwaita 1.x (latest stable) reference documentation under `docs/libadwaita/`. Automated agents MUST consult `docs/libadwaita/` for widget and helper guidance when making UI changes. These local docs are the canonical reference for UI implementation in this repo and are preferred over remote fetches to avoid network variability and version skew.
- General project documentation is in `docs/`; use `docs/` first for any implementation or styling questions before consulting upstream web pages.

- Repo entry & runtime
  - App entry: `src/main.rs`. A Tokio runtime is spawned on a background thread and a global handle is stored via `OnceLock`. Use `crate::runtime_handle()` to run async HTTP work on the background runtime.
  - UI run-loop: GTK / libadwaita run on the GLib main loop. Never touch GTK widgets from Tokio threads.

- Key modules (big picture)
  - `src/ui/` — all UI components and glue. `src/ui/main_window.rs` plus `src/ui/main_window/` submodules show most interaction patterns (loading repos, refreshing UI, connecting buttons).
  - `src/api/` — GitHub client, endpoints and models. See `src/api/client.rs` and `src/api/models.rs` for API surface.
  - `src/auth/` — OAuth Device Flow implementation (`device` module). Changes to auth must ensure TokenStorage interaction remains compatible.
  - `src/storage/token_storage.rs` — secure token handling via system keyring. This file contains a live test of the keyring; edits can affect developer machines.
  - `src/cache.rs`, `src/preferences.rs`, `src/favorites.rs` — app-level caching and persistence helpers used throughout the UI.

- Concurrency & integration patterns (important)
  - Background HTTP and long-running work: run on Tokio via `crate::runtime_handle().spawn(async move { ... })` (see `load_repositories`, `spawn_repo_status_tasks`).
  - UI updates: schedule UI changes on GLib using `glib::MainContext::default().spawn_local(...)` or `glib::idle_add_local_once(...)`. Follow the code in `src/ui/main_window.rs` as the canonical pattern.
  - Shared state: use `Arc<Mutex<T>>` (parking_lot::Mutex) for data shared between UI and background tasks. UI-specific ownership often uses `Rc<RefCell<...>>` for widgets/panes.

## Tokio + GTK runtime best practices (specific)

This project mixes a Tokio runtime for async HTTP work with GTK's GLib main loop. Follow these rules to avoid subtle deadlocks, panics, and non-Send/`'static` issues:

- Create a single, long-lived Tokio runtime (multi-threaded) and expose its `Handle` globally (the repo already uses `OnceLock<Handle>` in `src/main.rs`). Avoid creating multiple runtimes.
  - Example startup: `Builder::new_multi_thread().enable_all().worker_threads(num_cpus::get()).build()?` and store the handle.

- Never call `Runtime::block_on` or otherwise block the GLib main thread. Blocking the main thread freezes the UI and can deadlock Tokio resource drivers. If you need to run synchronous work, use `spawn_blocking` on Tokio or schedule it on GLib's thread pool.

- Run network and heavy async work on Tokio: from the GTK/main thread, dispatch work with the runtime handle and do NOT touch GTK objects inside those tasks.
  - Pattern:

```rust
let handle = crate::runtime_handle().clone();
handle.spawn(async move {
    let repos = client.list_repos().await; // HTTP on tokio
    // Marshal results back to the GLib main loop for UI updates
    glib::MainContext::default().spawn_local(async move {
        // safe to touch GTK widgets here
    });
});
```

- Avoid awaiting Tokio JoinHandles inside GLib async contexts. Instead either:
  - Await inside Tokio tasks and then call `spawn_local` to update UI; or
  - Use channels (oneshot/mpsc) to send results back and handle them on the GLib side.

- `tokio::spawn` requires futures to be `Send + 'static`. Any non-Send state (e.g., `Rc` or GTK objects) must not be moved into Tokio tasks. Use `Arc` for shared state or keep GTK references only on the GLib side.

- For non-Send async computations that must run on the main loop, use `glib::MainContext::default().spawn_local(...)` or `glib::source::spawn_local` — these run on GLib's executor and can safely use non-Send GTK types.

- For blocking CPU work, use `tokio::task::spawn_blocking` so the async runtime's IO threads aren't blocked.

- If you ever need to run tokio-owned code on the current thread (rare), use `Handle::enter()` carefully and only from non-GLib threads; avoid entering a Tokio runtime on the GLib main thread.

- Common pitfalls to watch for:
  - Holding GTK objects or `Rc<...>` across `.await` in a Tokio-spawned task (will fail `Send`/lifetime checks).
  - Calling `Runtime::block_on` on the main thread.
  - Creating short-lived runtimes repeatedly (costly and may leak OS threads if misused).

These specifics are drawn from Tokio and gtk-rs patterns — follow them when adding async logic or refactoring existing code.

- snapcraft.yaml API reference

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [makoni/actioneer-gtk](https://github.com/makoni/actioneer-gtk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
