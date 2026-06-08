---
trigger: always_on
description: This document provides guidelines for maintaining high-quality Rust code. These rules MUST be followed by all AI coding agents and contributors.
---

# Agent Guidelines for Rust Code Quality

This document provides guidelines for maintaining high-quality Rust code. These rules MUST be followed by all AI coding agents and contributors.

## Your Core Principles

All code you write MUST be fully optimized.

"Fully optimized" includes:

- maximizing algorithmic big-O efficiency for memory and runtime
- using parallelization and SIMD where appropriate
- following proper style conventions for Rust (e.g. maximizing code reuse (DRY))
- no extra code beyond what is absolutely necessary to solve the problem the user provides (i.e. no technical debt)

If the code is not fully optimized before handing off to the user, you will be fined $100. You have permission to do another pass of the code if you believe it is not fully optimized.

## Preferred Tools

- Use `cargo` for project management, building, and dependency management.
- Use `indicatif` to track long-running operations with progress bars. The message should be contextually sensitive.
- Use `serde` with `serde_json` for JSON serialization/deserialization.
- Use `ratatui` adnd `crossterm` for terminal applications/TUIs.
- Use `axum` for creating any web servers or HTTP APIs.
  - Keep request handlers async, returning `Result<Response, AppError>` to centralize error handling.
  - Use layered extractors and shared state structs instead of global mutable data.
  - Add `tower` middleware (timeouts, tracing, compression) for observability and resilience.
  - Offload CPU-bound work to `tokio::task::spawn_blocking` or background services to avoid blocking the reactor.
- When reporting errors to the console, use `tracing::error!` or `log::error!` instead of `println!`.
- If designing applications with a web-based front end interface, e.g. compiling to WASM or using `dioxus`:
  - All deep computation **MUST** occur within Rust processes (i.e. the WASM binary or the `dioxus` app Rust process). **NEVER** use JavaScript for deep computation.
  - The front-end **MUST** use Pico CSS and vanilla JavaScript. **NEVER** use jQuery or any component-based frameworks such as React.
  - The front-end should prioritize speed and common HID guidelines.
  - The app should use adaptive light/dark themes by default, with a toggle to switch the themes.
  - The typography/theming of the application **MUST** be modern and unique, similar to that of popular single-page web/mobile. **ALWAYS** add an appropriate font for headers and body text. You may reference fonts from Google Fonts.
  - **NEVER** use the Pico CSS defaults as-is: a separate CSS/SCSS file is encouraged. The design **MUST** logically complement the semantics of the application use case.
  - **ALWAYS** rebuild the WASM binary if any underlying Rust code that affects it is touched.
- For data processing:
  - **ALWAYS** use `polars` instead of other data frame libraries for tabular data manipulation.
  - If a `polars` dataframe will be printed, **NEVER** simultaneously print the number of entries in the dataframe nor the schema as it is redundant.
  - **NEVER** ingest more than 10 rows of a data frame at a time. Only analyze subsets of data to avoid overloading your memory context.
- If using Python to implement Rust code using PyO3/`maturin`:
  - Rebuild the Python package with `maturin` after finishing all Rust code changes.
  - **ALWAYS** use `uv` for Python package management and to create a `.venv` if it is not present. **NEVER** use the base system Python installation.
  - Ensure `.venv` is added to `.gitignore`.
  - Ensure `ipykernel` and `ipywidgets` is installed in `.venv` for Jupyter Notebook compatability. This should not be in package requirements.
  - **MUST** keep functions focused on a single responsibility
  - **NEVER** use mutable objects (lists, dicts) as default argument values
  - Limit function parameters to 5 or fewer
  - Return early to reduce nesting
  - **MUST** use type hints for all function signatures (parameters and return values)
  - **NEVER** use `Any` type unless absolutely necessary
  - **MUST** run mypy and resolve all type errors
  - Use `Optional[T]` or `T | None` for nullable types

## Code Style and Formatting

- **MUST** use meaningful, descriptive variable and function names
- **MUST** follow Rust API Guidelines and idiomatic Rust conventions
- **MUST** use 4 spaces for indentation (never tabs)
- **NEVER** use emoji, or unicode that emulates emoji (e.g. ✓, ✗). The only exception is when writing tests and testing the impact of multibyte characters.
- Use snake_case for functions/variables/modules, PascalCase for types/traits, SCREAMING_SNAKE_CASE for constants
- Limit line length to 100 characters (rustfmt default)
- Assume the user is a Python expert, but a Rust novice. Include additional code comments around Rust-specific nuances that a Python developer may not recognize.

## Documentation

- **MUST** include doc comments for all public functions, structs, enums, and methods
- **MUST** document function parameters, return values, and errors
- Keep comments up-to-date with code changes
- Include examples in doc comments for complex functions

Example doc comment:

````rust
/// Calculate the total cost of items including tax.
///
/// # Arguments
///

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [akiasprin/ripple-reader](https://github.com/akiasprin/ripple-reader) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-08 -->
