---
trigger: always_on
description: This document is for AI agents and contributors working on the marmite codebase itself. For building sites with marmite, see the embedded skill (`marmite --skill`).
---

# Marmite Development Guide

This document is for AI agents and contributors working on the marmite codebase itself. For building sites with marmite, see the embedded skill (`marmite --skill`).

## About

Marmite (**Mar**kdown **M**akes S**ite**s) is a minimal, fast static site generator written in Rust. It takes a folder of Markdown files, combines them with Tera templates, and produces a static HTML website. Single binary, zero runtime dependencies, zero-config by default.

- Repository: https://github.com/rochacbruno/marmite
- Site: https://marmite.blog

## Tech Stack

- **Language:** Rust
- **Templating:** [Tera](https://keats.github.io/tera/docs/) (Jinja2-like syntax)
- **Markdown:** [Comrak](https://docs.rs/comrak/) (CommonMark + GFM extensions)
- **CLI:** [Clap](https://docs.rs/clap/) (derive macros)
- **Syntax Highlighting:** [Arborium](https://docs.rs/arborium/) (tree-sitter based, build-time)
- **Dev Server:** [tiny_http](https://docs.rs/tiny_http/) with live reload via [tungstenite](https://docs.rs/tungstenite/) (WebSocket)
- **Task Runner:** [Mask](https://github.com/jacobdeichert/mask) (tasks defined in `maskfile.md`)
- **Image Processing:** [image](https://docs.rs/image/) crate with [rayon](https://docs.rs/rayon/) for parallel resizing
- **Embedded Assets:** [rust_embed](https://docs.rs/rust-embed/) (templates, static files, shortcodes, agent skills compiled into binary)

## Project Structure

```
src/
  main.rs             Entry point, CLI command routing
  cli.rs              Clap argument definitions
  config.rs           Marmite struct (marmite.yaml deserialization)
  content.rs          Content struct, frontmatter parsing, slug generation
  embedded.rs         Embedded assets (templates, static, shortcodes, agent skills)
  site.rs             Core site generation logic (~2000 lines)
  templates.rs        Template initialization and theme setup
  tera_functions.rs   Custom Tera functions (url_for, group, get_posts, etc.)
  tera_filter.rs      Custom Tera filters (default_date_format, remove_draft)
  shortcodes.rs       Shortcode processing with regex pattern matching
  parser.rs           Markdown to HTML conversion with comrak
  feed.rs             RSS feed generation
  gallery.rs          Image gallery processing
  highlight.rs        Build-time syntax highlighting
  image_provider.rs   Automatic banner image download (picsum)
  image_resize.rs     Parallel image resizing with incremental builds
  server.rs           Built-in HTTP server with WebSocket live reload
  theme_manager.rs    Remote theme download and installation
  re.rs               Shared regex patterns
  tests/              Unit tests (one file per module, calls code directly)

tests/                Integration tests (runs marmite as a subprocess via process::Command)

example/              Complete working example site - primary dev/test target
  marmite.yaml        Example configuration
  content/            Markdown files (posts, pages, _ prefixed fragments)
  templates/          Tera HTML templates
  static/             CSS, JS, fonts, colorschemes
  shortcodes/         Built-in shortcode definitions (Tera macros)
  theme_template/     Default theme scaffold used by --start-theme
  ai/llms.txt         LLM-readable documentation index

.agents/              Embedded agent skill files (compiled into binary via rust_embed)
  skills/marmite/
    SKILL.md           Main skill document with workflows
    references/        Detailed reference files (config, CLI, templates, etc.)
```

## Development Workflow

### First-time setup

```bash
# Install mask task runner (if not already installed)
cargo install mask

# Build and serve the example site with full trace logging and live reload
mask serve
```

The site runs at http://localhost:8000 with auto-rebuild on file changes. Edit source in `src/` or content in `example/` and it rebuilds automatically.

### Everyday commands

| Command | What it does |
|---------|-------------|
| `mask serve` | Build and serve the example site with live reload and full trace logging |
| `mask watch` | Watch for changes, rebuild without serving |
| `mask fmt` | Format code with `cargo fmt` |
| `mask check` | Check formatting + run clippy |
| `mask test` | Run all tests (unit + integration) |
| `mask test_unit` | Run unit tests only (`cargo test --bin marmite`) |
| `mask test_integration` | Run integration tests only (`cargo test --test '*'`) |
| `mask build` | Build release binary |
| `mask pedantic` | Run clippy with pedantic warnings |
| `mask fix` | Auto-apply clippy fixes |
| `mask pedantic_fix` | Auto-apply clippy pedantic fixes |

### Running specific tests

```bash
# A specific unit test by name
cargo test --bin marmite test_embedded_agent_skills

# A specific integration test file
cargo test --test basic_functionality

# All tests with output
cargo test -- --nocapture
```

### Serving with a theme

```bash
# Serve with the theme_template theme
mask serve_theme

# Serve the actual marmite.blog site locally
mask serve_site
```

## Code Conventions

- Follow `rustfmt` defaults. Always run `mask fmt` before committing.
- Follow `clippy` defaults. Always run `mask check` to verify.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rochacbruno/marmite](https://github.com/rochacbruno/marmite) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->
