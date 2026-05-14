---
trigger: always_on
description: `giv` is a Rust CLI tool and library for generating useful values (dates, UUIDs, keys, pi digits, random numbers, bytes, emoji, and special characters). Published on crates.io, it emphasizes strict code quality, comprehensive documentation, and safe coding practices.
---

# giv - Project Context for Claude

## Project Overview

`giv` is a Rust CLI tool and library for generating useful values (dates, UUIDs, keys, pi digits, random numbers, bytes, emoji, and special characters). Published on crates.io, it emphasizes strict code quality, comprehensive documentation, and safe coding practices.

The project provides both:

- **Binary**: A command-line tool (`giv`) for interactive use
- **Library**: Rust API for programmatic access to all generation functions

## Key Project Characteristics

### Strict Linting and Quality Standards

- **Forbidden unsafe code**: `unsafe_code = "forbid"`
- **Required documentation**: All items (public and private) must be documented
  - Missing docs are denied (both code and rustdoc)
  - All functions must include `# Errors`, `# Panics`, and `# Safety` docs where applicable
- **No direct stdout/stderr**: Use the `output` module instead
  - `print_stdout` and `print_stderr` are denied in clippy
  - All output goes through the structured output system

### Architecture Patterns

- **Command modules**: Named with `c_` prefix (e.g., `date`, `key`, `uuid`, `pi`, `rng`, `bytes`, `chars`)
- **Feature flags**: Each command is a Cargo feature that can be independently enabled/disabled
- **Output system**:
  - Core output trait in `app/output/output_trait.rs`
  - Each command has its own `output.rs` with a structured output type
  - Supports both plain text and JSON output via the `Output` trait
  - JSON output uses descriptive object properties (e.g., `{"key":"..."}`, `{"pi":"...","rounded":true}`)
  - Includes metadata fields where relevant (version, precision, rounding flags, source values)
- **Error handling**: Custom error types in `error.rs` using `thiserror`

### File Organization (One Item Per File)

- **Guideline**: Place one public item (struct, enum, or trait) per file as a general rule.
  - File names should match the item name (e.g., `Schema` struct goes in `schema.rs`)
  - Each file contains the item and all its implementations (Display, Default, methods, etc.)
  - Type aliases are exempt from this rule and can be grouped logically
  - When violating this guideline, include a comment explaining why:

    ```rust
    // Multiple error types grouped together for cohesion.
    pub enum AtsError { ... }
    pub enum LexError { ... }
    ```

- **Benefits of this pattern**:
  - Clear file-to-type mapping for navigation
  - Focused context when editing specific types
  - Precise git history (changes to `Field` only touch `field.rs`)
  - Reduced merge conflicts when working on different types
  - Tests can be colocated with their specific type

### Module Organization (mod.rs as Table of Contents)

`mod.rs` files should **only** contain module declarations and re-exports.

- All implementation code (structs, enums, functions, impls, tests) must be in separate files
- `mod.rs` serves as the module's table of contents
- Each public item gets its own file following the "one item per file" convention

**Example**:

```rust
// config/mod.rs - GOOD: Only declarations and re-exports
pub mod configuration;
pub mod site;
pub mod source;

pub use configuration::Configuration;
pub use site::SiteConfig;
pub use source::SourceConfig;
```

**Benefits**: Clear module index, easier navigation, better git history, consistent with "one item per file".

### Code Structure

```text
src/
├── main.rs           # Entry point with command routing
├── error.rs          # Error types
├── app/              # Application infrastructure
│   ├── cli/          # Clap-based CLI definitions
│   ├── output/       # Output trait and formatting
│   └── context.rs    # Command execution context
├── bytes/          # Random byte generation
│   ├── mod.rs
│   └── output.rs     # BytesOutput struct
├── chars/          # Emoji and special character conversion
│   ├── mod.rs
│   ├── patterns.rs   # Character pattern conversion
│   └── output.rs     # CharsOutput struct
├── date/           # Date/time generation
│   ├── mod.rs
│   ├── date_format.rs
│   ├── date_kind.rs
│   └── output.rs     # DateOutput struct
├── key/            # Random key generation
│   ├── mod.rs
│   └── output.rs     # KeyOutput struct
├── pi/             # Pi digit generation
│   ├── mod.rs
│   ├── decimals.rs   # Pre-calculated pi digits
│   └── output.rs     # PiOutput struct
├── rng/            # Random number generation
│   ├── mod.rs
│   ├── spec.rs       # Specification parsing
│   ├── execute.rs    # Execution logic
│   ├── generator.rs  # RNG functions
│   ├── result.rs     # Result types
│   └── output.rs     # RngOutput struct
└── uuid/           # UUID v7 generation
    ├── mod.rs
    └── output.rs     # UuidOutput struct
```

### Development Workflow

- Install from crates.io: `cargo install giv`
- Build/install locally: `cargo install --path .`
- Run locally (binary requires `bin` feature): `cargo run --features="bin" -- --version`
- Test: `cargo test`
- Lint: `cargo clippy`
- Documentation: `cargo doc --open`

### Library Usage


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [theroyalwhee0/giv](https://github.com/theroyalwhee0/giv) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
