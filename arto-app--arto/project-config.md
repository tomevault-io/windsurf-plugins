---
trigger: always_on
description: > **📖 For detailed best practices and tips:** See [TIPS.md](.claude/TIPS.md)
---

# Project-Specific Rules

> **📖 For detailed best practices and tips:** See [TIPS.md](.claude/TIPS.md)

## Quick Reference

- **Code Comments**: Must be in English
- **Test Code**: Use `indoc` crate for multi-line strings
- **Module System**: Use Rust 2018+ (no `mod.rs`)
- **Icon Management**: Use `add-icon` skill
- **UI/UX Design**: See `.claude/rules/ui-design.md`
- **Quality Check**: Run `just fmt check test` before reporting completion
- **Application Launch**: Do NOT launch the application; the user handles this

## Development Workflow

### Quality Assurance

**CRITICAL: Before reporting task completion, ALWAYS run:**

```bash
just fmt check test
```

This command runs:
- `cargo fmt` + `oxfmt` - Code formatting (Rust + TypeScript/CSS)
- `cargo clippy` + `oxlint` - Linting and best practices
- `cargo test` - All tests

**Do NOT report completion if any of these fail.** Fix all issues first.

## Content Source

**Always check existing content files before writing descriptions:**

- Welcome page content: `assets/welcome.md`
- README: Project description and philosophy
- Use actual project descriptions, not generic placeholders

## Architecture Patterns

### Single-Instance Architecture

**Arto enforces single-instance via IPC (Inter-Process Communication):**

When a new Arto process launches:
1. Check for existing instance via Unix domain socket connection
2. If found → send paths to existing instance via JSON Lines protocol, exit(0)
3. If not found → become primary instance, start IPC server for future launches

**Protocol (JSON Lines):**
```json
{"type":"file","path":"/path/to/file.md"}
{"type":"directory","path":"/path/to/dir"}
{"type":"reopen"}
```

**Implementation:** See `desktop/src/ipc.rs` for detailed architecture documentation.

**Why single-instance:** Prevents multiple processes from conflicting over file watches, configuration writes, and persisted state management.

### Window Management & Lifecycle

**Within the single instance, Arto uses a multi-window architecture:**

#### Window Types

1. **Main Windows** (user-visible)
   - First window launched from `main()` uses MainApp component
   - Handles system events: file open, app reopen
   - Uses `WindowCloseBehaviour::WindowHides` (last window hides instead of quitting)
   - Additional windows created on demand (File → New Window)
   - Each has independent tabs and state

2. **Child Windows** (specialized)
   - Mermaid diagram viewer, etc.
   - Owned by a parent main window
   - Auto-close when parent closes

#### Window Creation Pattern

```rust
// First window (synchronous initialization in main())
let is_first_window = true;
let theme = window::settings::get_theme_preference(is_first_window);
let directory = window::settings::get_directory_preference(is_first_window);
let sidebar = window::settings::get_sidebar_preference(is_first_window);

// Launch MainApp with pre-resolved preferences
dioxus::LaunchBuilder::desktop()
    .with_cfg(config)
    .launch(components::main_app::MainApp);

// Additional windows (async creation)
window_manager::create_new_main_window(file, directory, show_welcome);
```

**Key differences:**
- **First window**: Resolved synchronously in `main()` before Dioxus launch (eliminates flash)
- **Additional windows**: Created asynchronously using helper functions
- **Startup**: Uses `PersistedState` from `state.json` (last closed window)
- **New Window**: Accesses last focused window's `AppState` directly via `WINDOW_STATES` mapping

#### Window Lifecycle Hooks

```rust
// In App component - register state on creation
let mut state = use_context_provider(|| {
    let mut app_state = AppState::new(theme);
    // ... initialization ...
    crate::window::register_window_state(window().id(), app_state);
    app_state
});

// Cleanup on window close
use_drop(move || {
    // Unregister from WINDOW_STATES mapping
    crate::window::unregister_window_state(window_id);

    // Save state on window close
    let mut persisted = PersistedState::from(&state);

    // Capture window metrics for restoration
    let window_metrics = crate::window::metrics::capture_window_metrics(&window().window);
    persisted.window_position = window_metrics.position;
    persisted.window_size = window_metrics.size;

    // Save to disk (synchronous, blocking)
    persisted.save();

    // Close child windows owned by this window
    crate::window::close_child_windows_for_parent(window_id);
});
```

**IMPORTANT:** The `save()` method is synchronous and blocking, suitable for `use_drop()` context.

### State Management Hierarchy

**Three-tier system (see TIPS.md and architecture-overview.md for details):**

1. **Global Statics** - Shared across windows (CONFIG, WINDOW_STATES mapping, broadcast channels)
2. **Context (AppState)** - Per-window state (tabs, active tab, zoom)
3. **Local (use_signal)** - Component-only UI state

**Startup priority:**
1. `PersistedState` from `state.json` (last closed window)
2. Fallback to `Config` defaults

**New window priority:**
1. Last focused window's `AppState` via `WINDOW_STATES` mapping
2. Fallback to `PersistedState::load()` then `Config` defaults

### Configuration System

**Dual-file system (see TIPS.md and architecture-overview.md for details):**

```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [arto-app/Arto](https://github.com/arto-app/Arto) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
