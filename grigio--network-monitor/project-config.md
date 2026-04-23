---
trigger: always_on
description: - **Rust 2021 Edition**: Systems programming with memory safety
---

# Network Monitor - Rust + GTK4 + TUI Guide

## Technology Stack
- **Rust 2021 Edition**: Systems programming with memory safety
- **GTK4**: Modern cross-platform GUI framework with v4_14 features
- **Libadwaita**: GNOME-style UI components with v1_5 features
- **Ratatui**: Terminal User Interface framework (beta version for latest features)
- **Crossterm**: Cross-platform terminal handling with event streaming
- **Native socket parsing**: Direct `/proc/net` filesystem access
- **Inode-based process mapping**: Socket-to-process identification

## Project Structure
```
src/
├── main.rs          # GTK4 application entry point
├── tui_main.rs      # TUI application entry point
├── ui/              # GTK4 UI components and widgets
├── models/          # Data structures and state (shared)
├── services/        # Business logic and system calls (shared)
├── utils/           # Helper functions (shared)
├── error.rs         # Custom error types with thiserror
└── error_tests.rs    # Error handling tests
```

## Performance Optimizations

### UI Performance Enhancements
- **Debouncing**: Implemented debouncing for UI updates (200ms delay, 500ms minimum interval) to prevent excessive updates and reduce CPU usage
- **Virtualization**: Added row virtualization for large datasets (>100 connections) to show only first/last 50 rows with placeholders, significantly reducing widget count
- **Column Width Caching**: Implemented caching system for column widths with 10px threshold changes to avoid repeated layout calculations
- **Layout Optimization**: Reduced sampling rate for column width measurement (1 every 5 rows) and added conservative text width estimation

### TUI Performance
- **Layout Caching**: Added layout cache system with validation based on width and connection count changes
- **Skip Rendering**: Implemented skip rendering when no significant changes detected to improve TUI responsiveness
- **FPS Monitoring**: Added FPS monitoring with warnings for low performance

### Critical Implementation Notes
- **RefCell Management**: Careful RefCell borrowing patterns implemented to avoid runtime panics. Multiple mutable RefCell accesses are properly scoped to prevent borrowing conflicts.
- **Memory Management**: Widget reuse implemented to minimize memory allocation and improve performance during frequent updates.
- **Thread Safety**: All UI updates properly scheduled through GTK's main loop using `glib::spawn_future_local()` and `glib::idle_add_local_once()`.

## Common Patterns

### Actions and Menus (GTK4 Guidelines)
**IMPORTANT**: Always follow the official GTK4 actions and menus documentation at https://gtk-rs.org/gtk4-rs/stable/latest/book/actions.html#menus for implementing menus and actions.

```rust
use gio::ActionEntry;

// Use ActionEntry builder pattern for actions
let action_about = ActionEntry::builder("about")
    .activate(move |window: &ApplicationWindow, _, _| {
        // Action implementation
    })
    .build();

// Add actions using add_action_entries method
window.add_action_entries([action_about]);

// Set keyboard accelerators
app.set_accels_for_action("win.about", &["F1"]);

// Create menu model with proper action references
let menu = Menu::new();
let section = Menu::new();
section.append(Some("About"), Some("win.about"));
menu.append_section(Some("Help"), &section);
```

**Key Requirements**:
- Use `ActionEntry::builder()` instead of `SimpleAction::new()`
- Use `add_action_entries()` instead of individual `add_action()` calls
- Organize actions with proper prefixes: `app.*` for application-level, `win.*` for window-level
- Set keyboard accelerators with `set_accels_for_action()`
- Reference actions in menus using full action name (e.g., `"win.about"`)

**Menu Styling (Adwaita)**:
```css
/* Fix double borders and transparency */
popover {
    border: none;
    box-shadow: 0 4px 12px alpha(black, 0.12);
    background: var(--popover-bg-color);
}

popover contents {
    border: none;
    box-shadow: none;
    background: transparent;
}

menubutton > popover {
    border: none;
    box-shadow: 0 4px 12px alpha(black, 0.12);
}
```

### Async Operations with GTK
```rust
use glib::clone;
use tokio::runtime::Runtime;

// Spawn async task from GTK callback
let rt = Runtime::new().unwrap();
glib::spawn_future_local(async move {
    rt.spawn(async {
        // Async system calls here
    });
});
```

### State Management
- Use `Rc<RefCell<T>>` for shared mutable state
- Consider `once_cell::sync::Lazy` for global state
- Implement `Default` and `Clone` for complex types

## Development Commands
```bash
# Build and run
cargo run                    # GTK4 version
cargo run --bin nmt          # TUI version
cargo build --release        # Release build

# Code quality
cargo fmt                    # Format code
cargo clippy -- -D warnings  # Lint with strict warnings
cargo test                   # Run tests

# Dependency management
cargo update                 # Update dependencies
cargo outdated               # Check for outdated dependencies
cargo audit                  # Security audit
cargo deny check             # License and dependency checks

# Installation
./scripts/install.sh         # Local install
sudo ./scripts/install.sh    # System-wide install

# before commit run
cargo outdated

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [grigio/network-monitor](https://github.com/grigio/network-monitor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
