---
trigger: always_on
description: Guidance for LLM-based code agents
---

Guidance for LLM-based code agents
==================================

This file provides guidance to LLM-based code agents (e.g., Claude Code,
OpenCode) when working with code in this repository.


Project overview
----------------

Hongdown is a Markdown formatter that enforces Hong Minhee's Markdown style
conventions.  The formatter is implemented in Rust (Edition 2024) using the
comrak library for parsing.  It produces consistently formatted Markdown
output following a distinctive style used across multiple projects including
Fedify, LogTape, and Optique.

### Project architecture

~~~~ text
hongdown/
├── src/
│   ├── main.rs           # CLI entry point (clap-based argument parsing)
│   ├── lib.rs            # Library entry point (Options, format functions)
│   ├── config.rs         # Configuration file handling (.hongdown.toml)
│   ├── wasm.rs           # WASM bindings (wasm-bindgen)
│   └── serializer/       # Core formatting logic
│       ├── mod.rs        # Main serializer module
│       ├── state.rs      # Serializer state and types
│       ├── block.rs      # Block quotes and GitHub alerts
│       ├── code.rs       # Code blocks (fenced and indented)
│       ├── document.rs   # Document-level handling
│       ├── escape.rs     # Text escaping utilities
│       ├── inline.rs     # Inline elements (emphasis, code spans)
│       ├── link.rs       # Links and images
│       ├── list.rs       # Ordered and unordered lists
│       ├── table.rs      # Table formatting
│       ├── wrap.rs       # Text wrapping utilities
│       └── tests.rs      # Unit tests for serializer
├── tests/
│   └── integration.rs    # Integration tests
├── packages/
│   ├── hongdown/         # CLI npm package (hongdown)
│   │   ├── package.json
│   │   └── bin/hongdown.js
│   └── wasm/             # WASM library package (@hongdown/wasm)
│       ├── src/          # TypeScript wrapper source
│       └── test/         # Tests for Node.js, Bun, Deno
├── demo/                 # Web demo application (hongdown-demo)
│   ├── src/
│   │   ├── components/   # UI components (TabBar, Options, Inputs)
│   │   ├── App.tsx       # Main application logic
│   │   ├── index.tsx     # Entry point
│   │   ├── sample.ts     # Sample Markdown content
│   │   └── styles.css    # Custom styles (scrollbar, etc.)
│   ├── index.html        # HTML template
│   ├── vite.config.ts    # Vite configuration
│   └── uno.config.ts     # UnoCSS configuration
├── pnpm-workspace.yaml   # pnpm workspace configuration
└── package.json          # Root workspace package.json
~~~~


Development commands
--------------------

This is a Rust project using Cargo as the build system.  We use [mise] for
task management to streamline common development workflows.

[mise]: https://mise.jdx.dev/

### Initial setup

After cloning the repository, set up the Git pre-commit hook to automatically
run quality checks before each commit:

~~~~ bash
mise generate git-pre-commit --task=check --write
~~~~

### Building

~~~~ bash
cargo build            # Debug build
cargo build --release  # Release build
~~~~

### Testing

The recommended way to run all tests is using mise:

~~~~ bash
mise run test              # Run all tests (Rust + WASM)
mise run test:rust         # Run Rust tests only
mise run test:wasm         # Run WASM package tests only
~~~~

You can also run Cargo tests directly:

~~~~ bash
cargo test                        # Run all Rust tests
cargo test <name>                 # Run tests matching a name pattern
cargo test serialize_table        # e.g., runs all table serialization tests
cargo test -- --nocapture         # Show println! output
cargo test -- --test-threads=1    # Run tests sequentially
~~~~

Tests are located in several places:

 -  *Rust unit tests*: *src/serializer/tests.rs* contains serializer tests
 -  *Rust integration tests*: *tests/integration.rs* contains CLI and full
    document tests
 -  *WASM package tests*: *packages/wasm/test/* contains tests that run on
    Node.js, Bun, and Deno

Test helper functions in *src/serializer/tests.rs*:

 -  `parse_and_serialize(input)` - Format with default options
 -  `parse_and_serialize_with_options(input, options)` - Format with custom
    options
 -  `parse_and_serialize_with_source(input)` - Format with source context
    for directive support
 -  `parse_and_serialize_with_warnings(input)` - Format and capture warnings

### Demo application

The *demo/* directory contains a web-based playground for Hongdown.  To run
the demo locally:

~~~~ bash
pnpm --filter hongdown-demo dev     # Start development server
pnpm --filter hongdown-demo build   # Build for production
~~~~

The demo app uses:

 -  *Solid.js* for reactive UI
 -  *UnoCSS* for styling with atomic CSS
 -  *Vite* for bundling and development
 -  *@hongdown/wasm* for real-time Markdown formatting

Design principles:

 -  *Borderless design*: Visual hierarchy is established through background
    color differences (e.g., `bg-neutral-50` vs `bg-white` in light mode)
    rather than borders
 -  *Responsive layout*: Desktop shows split-view (editor/output side-by-side),
    mobile shows tabbed interface (Editor/Output/Warnings tabs)
 -  *Dark mode*: Automatically respects system preferences via
    `@media (prefers-color-scheme: dark)`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dahlia/hongdown](https://github.com/dahlia/hongdown) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
