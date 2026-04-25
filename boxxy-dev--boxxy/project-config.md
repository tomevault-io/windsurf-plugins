---
trigger: always_on
description: Manages the split-pane terminal environment. Features a deep modular architecture (`pane/`) handling UI, gestures, events, media previews, and Claw integration.
---

# Boxxy-Terminal Agents & Architecture

## Observability & Debugging

Boxxy provides deep, real-time visibility into LLM interactions for developers and power users:
- **Cumulative Token Analytics:** Every session tracks input, output, and total tokens, persisted in `boxxy-db` and re-rendered during session resumption.
- **Dedicated Context Debugging:** To audit the exact payloads (System Prompt, History, Tools) sent to models, run the application with:
  ```bash
  BOXXY_DEBUG_CONTEXT=1 cargo run
  ```
  This is a strictly opt-in, high-signal logging mode that works in both debug and release builds (using `log::info!` on the `model-context` target), ensuring zero noise during standard development.
- **Visual Sidebar Logs:** The Claw sidebar acts as a read-only debug log, rendering tool results, process lists, and agent reasoning steps as structured UI components.

## Technology Stack
- **Language:** Rust 2024 (v1.94+)
- **Concurrency:** tokio v1.50, async-channel v2.3
- **UI Toolkit:** GTK 4.22 + Libadwaita 1.90 (via `gtk4-rs` 0.11)
- **Terminal Engine:** Custom async-first lock-free ANSI state machine absorbed directly into the `boxxy-vte` crate — no dependency on `libvte` or any C terminal library.

## Coding Guidelines

### 1. Rust Code Style
- **Formatting:** Always format with `cargo fmt --all` after writing or editing Rust files to ensure consistency across the workspace.
- **Enforcement:** Code must pass `cargo fmt --all -- --check` with zero diff. This project strictly follows the Rust 2024 style guidelines as configured in `rustfmt.toml`.
- **Inline Comments:** Use short, concise inline comments to explain *why* a particular approach was taken, especially for complex async flows, GTK property bindings, or state machine transitions. Avoid over-commenting obvious syntax. Maintain existing comments, updating them only if they become obsolete.

### 2. Modularity & Scoping
- **File Length Limit:** Keep source files under **700 lines of code** whenever possible. If a file exceeds this, refactor it into smaller, well-scoped modules.
- **Structural Integrity:** Use directory-based modules (e.g., `pane/mod.rs`) for complex components. Avoid monolithic `lib.rs` files; use them primarily for module declarations and public API exports.

### 3. UI & Resource Management
- **Declarative UI:** Do NOT write massive XML strings inside Rust source code. Always extract GTK widget trees into `.ui` XML files located in the `resources/ui/` directory.
- **GTK CSS Specificity:** Avoid using `!important` in CSS files. GTK's CSS engine does not reliably support it; instead, use higher selector specificity (e.g., `tabbar tab.color` instead of just `.color`) to override default styles.
- **GResource Integration:** Register all `.ui`, `.md` (prompts), `.css`, and icon files in `resources/resources.gresource.xml`. Load them at runtime via `gtk::Builder::from_resource` or `gio::resources_lookup_data`.
- **Build Automation:** Ensure all resources are tracked in `crates/app/build.rs` for automatic recompilation.

### 4. Concurrency & Performance
- **Non-Blocking UI:** Never perform synchronous disk I/O, heavy parsing, or network calls on the main GTK thread. Use `glib::spawn_future_local` and delegate heavy tasks to the global Tokio runtime.
- **RefCell Safety:** Avoid holding `RefCell` borrows across `.await` points to prevent runtime panics.

### 5. Technical Integrity & Testing
- **Database Logic:** ALL database operations MUST have corresponding unit tests in their respective crates (using `Db::new_in_memory()`). NEVER assume a query is correct without verification.
- **Memory Hygiene:** The "Background Observer" must be strictly tuned to exclude transient state. Implicit memories default to `unverified` and require manual promotion in `MEMORY.md`.
- **Validation:** Changes to core engine logic must be verified via `cargo test` and, where applicable, by adding a new YAML scenario to `scenario-runner`.

## Component Responsibilities

### 1. `boxxy-app` (Binary Crate)
Entry point. Initializes GTK/Libadwaita, registers GResources, bootstraps the main window, and ensures clean process termination.

### 2. `boxxy-window` (Library Crate)
Main UI Orchestrator using a modular MVU pattern. Manages the `AdwApplicationWindow`, tabs, and global state routing via `state.rs`, `ui.rs`, and the `update/` module. Acts as the primary orchestrator for the **Global Workspace Radar**, ensuring peer-to-peer agent discovery and global intent propagation across all windows. Respects the **"Lightweight First"** configuration by initializing agents in an inactive state unless `claw_on_by_default` is enabled. It monitors the active terminal pane to reflect its specific Claw mode state.

### 3. `boxxy-terminal` (Library Crate)
Manages the split-pane terminal environment. Features a deep modular architecture (`pane/`) handling UI, gestures, events, media previews, and Claw integration.

### 4. `boxxy-agent` (Binary/Library Crate)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [boxxy-dev/boxxy](https://github.com/boxxy-dev/boxxy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
