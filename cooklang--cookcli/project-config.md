---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

CookCLI is a command-line interface for managing Cooklang recipes. It's written in Rust and includes a web server with server-side rendered HTML using Askama templates and Tailwind CSS. The project follows UNIX philosophy where each command does one thing well.

## Build and Development Commands

### Building
```bash
# Full release build (includes CSS compilation)
make release

# Development build
cargo build
make build  # Includes CSS compilation

# Build CSS only
make css
npm run build-css

# Build JS only
make js
npm run build-js

# Build specific workspace member
cargo build -p cookcli

# Run without building
cargo run -- [command] [args]
```

### Testing
```bash
# Run all tests
cargo test

# Run specific test
cargo test test_name

# Run tests with output
cargo test -- --nocapture

# Run with debug logging
RUST_LOG=debug cargo run -- [command]
RUST_LOG=trace cargo run -- [command]  # More verbose
```

### Web UI Development
```bash
# Start development server with CSS built
make dev_server

# Start development with CSS watch mode
make css-watch  # In one terminal
cargo run -- server ./seed  # In another terminal

# Install dependencies (first time setup)
npm install

# Build CSS for production
npm run build-css

# Watch CSS changes during development
npm run watch-css

# Build JS for production
npm run build-js

# Watch JS changes during development
npm run watch-js
```

### Linting and Formatting
```bash
cargo fmt         # Format code
cargo clippy      # Lint code
```

## Architecture

### Workspace Structure
This is a Cargo workspace with multiple crates:
- `cookcli` (this crate) - The CLI application
- `../cooklang-rs` - Core Cooklang parser and models
- `../cooklang-find` - Recipe discovery and search
- `../cooklang-reports` - Template-based report generation
- `../cooklang-import` - Recipe import from websites (external dependency)

### Command Architecture
Each command is a module in `src/` with a consistent pattern:
1. `XxxArgs` struct in `src/args.rs` or module file - Clap argument definitions
2. `pub fn run(ctx: &Context, args: XxxArgs) -> Result<()>` - Entry point
3. Commands receive a `Context` with base_path, aisle, and pantry configuration

### Context System
The `Context` struct (in `src/main.rs`) provides:
- `base_path`: Current working directory for recipes
- `aisle()`: Returns path to aisle.conf (local `./config/` or global `~/.config/cook/`)
- `pantry()`: Returns path to pantry.conf (same search pattern)

Configuration search order:
1. `./config/[aisle.conf|pantry.conf]` - Local to recipe directory
2. `~/Library/Application Support/cook/` (macOS) or `~/.config/cook/` (Linux)

### Command Modules

#### Core Commands
- `recipe`: Parse and display recipes (supports multiple output formats)
- `shopping_list`: Generate shopping lists with ingredient aggregation
- `server`: Web server using Axum + Askama templates with Tailwind CSS
- `search`: Full-text recipe search
- `import`: Import from websites via cooklang-import
- `doctor`: Validate recipes and check configurations
- `seed`: Initialize with example recipes
- `report`: Generate custom outputs using Jinja2 templates
- `update`: Self-update the CookCLI binary to the latest version (can be disabled with --no-self-update feature)

#### Utility Modules
- `util/`: Shared utilities for parsing, conversion, and output formatting
  - Recipe parsing and scaling
  - Output format conversions (human, JSON, YAML, Markdown, Cooklang)
  - Shopping list generation with aisle/pantry support

### Web Server
- Backend: Axum web framework in `src/server/`
- Frontend: Server-side rendered HTML using Askama templates
- Templates: Located in `templates/` directory
- Styling: Tailwind CSS with custom components
- API handlers in `src/server/handlers/`
- Static files served from `static/` directory
- Shopping list stored as tab-delimited files in `/tmp/`

### Recipe Processing Pipeline
1. Recipe discovery via `cooklang-find` (handles paths and search)
2. Parsing via `cooklang` parser (lenient mode for validation)
3. Scaling and conversion in `util/` modules
4. Output formatting based on requested format

### Error Handling
- Uses `anyhow::Result` throughout for error propagation
- User-friendly error messages with context
- Special handling for recipe validation (warnings vs errors)

## Important Implementation Details

### Recipe Scaling
- Scaling notation: `recipe.cook:2` (uses colon)
- Handled in `util::split_recipe_name_and_scaling_factor()`
- Applied during parsing before output

### Shopping List Aggregation
- Ingredients with same name are automatically combined
- Unit conversion handled by cooklang crate
- Aisle categorization via aisle.conf
- Pantry filtering via pantry.conf (TOML format with quantities and dates)

### Doctor Command
- `validate`: Checks syntax, deprecated features, and recipe references
- `aisle`: Finds ingredients not in aisle configuration
- Exit codes: Normal (0) or strict mode (1) for CI/CD

### Report Command
- Uses cooklang-reports with Jinja2 templates
- Config builder pattern for scale, datastore, aisle, pantry

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cooklang/cookcli](https://github.com/cooklang/cookcli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
