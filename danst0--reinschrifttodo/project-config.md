---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Reinschrift is a multi-platform todo application:
- **Desktop GUI**: Native Rust + libadwaita (GTK4) application for GNOME
- **CLI**: Command-line interface for terminal workflows
- **Web**: Python Flask application with Docker support

Todos are stored as plain Markdown files, making them version-controllable and editor-agnostic. Supports WebDAV/Nextcloud synchronization.

## Project Structure (Cargo Workspace)

```
reinschrift/
├── Cargo.toml              # Workspace root
├── core/                   # Shared library (reinschrift-core)
│   ├── Cargo.toml
│   └── src/
│       ├── lib.rs          # Module declarations and re-exports
│       ├── data.rs         # Backward-compatible re-exports from modules
│       ├── types.rs        # TodoItem, TodoKey, DEFAULT_DUE_TIME
│       ├── config.rs       # BackendConfig, path management
│       ├── webdav.rs       # WebDAV client operations
│       ├── storage.rs      # Unified read/write abstraction
│       ├── parser.rs       # Regex patterns, parse_line, extract_title
│       ├── renderer.rs     # render_line, rewrite_line, rewrite_due
│       ├── todo.rs         # Business logic (load, toggle, add, delete)
│       ├── util.rs         # String helpers, marker generation
│       ├── i18n.rs         # Internationalization
│       ├── sorting.rs      # Sorting functions
│       ├── preferences.rs  # User preferences
│       └── i18n/           # Translation JSON files
├── gui/                    # GTK application (reinschrift-gui)
│   ├── Cargo.toml
│   └── src/
│       ├── main.rs         # GUI entry point
│       └── ui.rs           # GTK4/libadwaita UI
├── cli/                    # CLI application (reinschrift-cli)
│   ├── Cargo.toml
│   └── src/
│       ├── main.rs         # CLI entry point (clap)
│       ├── commands.rs     # Command implementations
│       └── output.rs       # Colored terminal output
└── webapp/                 # Web application (Python Flask)
```

## Build Commands

### Workspace (all Rust crates)
```bash
# Build entire workspace
cargo build --workspace --release

# Check entire workspace
cargo check --workspace

# Build specific crate
cargo build -p reinschrift-gui --release
cargo build -p reinschrift-cli --release
```

### GUI Application
```bash
# Development build & run
cargo run -p reinschrift-gui --release

# With custom database path
TODOS_DB_PATH=/path/to/db.md cargo run -p reinschrift-gui

# With custom language
cargo run -p reinschrift-gui -- --language en
```

### CLI Application
```bash
# Run CLI
cargo run -p reinschrift-cli -- list
cargo run -p reinschrift-cli -- add "New task +project @context"
cargo run -p reinschrift-cli -- done ^marker

# After build, binaries are at:
# - target/release/reinschrift_todo (GUI)
# - target/release/reinschrift (CLI)
```

### CLI Command Reference
```
reinschrift [OPTIONS] <COMMAND>

OPTIONS:
    -d, --database <PATH>    Path to markdown database
    -l, --language <LANG>    Language (de, en, es, fr, ja, sv)
    -j, --json               JSON output for scripting
        --no-color           Disable colored output

COMMANDS:
    list (ls)       List todos with filtering/sorting
    add             Add a new todo
    edit            Edit an existing todo
    delete (rm)     Delete todo(s)
    done            Mark as completed
    undone          Mark as incomplete
    today           Set due date to today
    tomorrow        Set due date to tomorrow
    search          Search todos
    config          WebDAV and AI configuration
```

### Web Application (Flask)
```bash
cd webapp

# Docker deployment
docker-compose up --build
# Access at http://localhost:5000

# Local development
python run.py
# Or: flask run

# Run tests
pip install -r requirements-dev.txt
pytest tests/ -v

# Type checking
mypy app/
```

### Git Hooks Setup
```bash
git config core.hooksPath .githooks
```

## Architecture

### Core Library (core/)

Modular design with focused responsibilities:

- **types.rs**: Core data structures
  - `TodoItem`, `TodoKey` structs with Serialize support
  - `DEFAULT_DUE_TIME` constant
- **config.rs**: Configuration management
  - `BackendConfig` enum (Local/WebDav)
  - Path management (`todo_path`, `set_todo_path`)
- **webdav.rs**: WebDAV client operations
  - URL construction, Nextcloud fallback logic
  - `test_webdav_connection`, read/write operations
- **storage.rs**: Unified storage abstraction
  - `read_content`, `write_content`, `get_fingerprint`
  - Dispatches to local filesystem or WebDAV
- **parser.rs**: Markdown parsing
  - Regex patterns for: +projects, @contexts, due:dates, ^IDs, rec:recurrence, ~note:"text"
  - `parse_line`, `extract_title`, `find_line_by_marker`
- **renderer.rs**: Line rendering
  - `render_line`, `rewrite_line`, `rewrite_due`
  - Completion marker handling
- **todo.rs**: Business logic
  - `load_todos`, `toggle_todo`, `add_todo`, `delete_todo`
  - Due date operations (`set_due_today/tomorrow/weekend/sometime`)
  - `next_due_date` for recurrence
- **util.rs**: String utilities
  - `generate_marker`, `encode_base36`
  - Note escaping/unescaping, token normalization

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [danst0/ReinschriftTodo](https://github.com/danst0/ReinschriftTodo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
