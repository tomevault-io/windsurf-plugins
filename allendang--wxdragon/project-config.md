---
trigger: always_on
description: wxDragon is a cross-platform GUI library that provides safe Rust bindings for wxWidgets. It enables developers to build native desktop applications for Windows, macOS, and Linux with a single Rust codebase.
---

# Pull Request Review Instructions for wxDragon

## Project Overview

wxDragon is a cross-platform GUI library that provides safe Rust bindings for wxWidgets. It enables developers to build native desktop applications for Windows, macOS, and Linux with a single Rust codebase.

**Key technologies:**

- Rust (edition 2024)
- wxWidgets 3.3.1 (C++ GUI framework)
- FFI bindings via wxdragon-sys
- Procedural macros via wxdragon-macros

## Critical Review Areas

### 1. Memory Safety & FFI Boundaries

**Priority: Critical**

- **Unsafe Code Audit**: All `unsafe` blocks must be carefully reviewed
  - Verify proper null pointer checks before dereferencing
  - Ensure lifetimes are correctly managed across FFI boundaries
  - Check that raw pointers from wxWidgets are valid before use
  - Confirm proper cleanup/destruction of wxWidgets objects

- **Resource Management**:
  - wxWidgets objects must be properly destroyed (check Drop implementations)
  - Event handlers should not create memory leaks or dangling references
  - Clone implementations should properly increment reference counts if needed

- **String Conversions**:
  - Verify proper UTF-8 handling when converting between Rust and wxWidgets strings
  - Check for potential panics in string conversions
  - Ensure CString conversions handle interior nulls appropriately

### 2. Cross-Platform Compatibility

**Priority: Critical**

wxDragon supports Windows (MSVC & MinGW), macOS, and Linux (GTK+). Review PRs for:

- **Platform-Specific Code**:
  - Check `#[cfg(target_os = "...")]` attributes are correct
  - Verify platform-specific implementations maintain API consistency
  - Test that examples build on all platforms (see CI workflow)

- **Toolchain Compatibility**:
  - Windows MinGW builds MUST use WinLibs GCC 15.1.0 UCRT
  - Ensure no hard-coded paths or platform-specific assumptions
  - Cross-compilation support (macOS ? Windows) should not be broken

- **CI Requirements**:
  - All platforms must pass: `build-linux`, `build-macos`, `build-windows-msvc`, `build-i686-pc-windows-msvc`, `build-msys-mingw64`
  - Verify CI workflow changes don't break any platform

### 3. API Design & Ergonomics

**Priority: High**

- **Builder Pattern Consistency**:
  - New widgets should follow the established builder pattern
  - Methods should be named `with_*` for builder chains
  - Example: `Widget::builder().with_title("...").with_size(...).build()`

- **Type Safety**:
  - Prefer strongly-typed enums over raw integers
  - Use bitflags for flag combinations (see SizerFlag)
  - Avoid exposing raw FFI types in public APIs

- **Event Handling**:
  - Event callbacks should be safe and ergonomic
  - Check for proper closure capture and lifetime management
  - Verify event handler macros work correctly

- **Naming Conventions**:
  - Follow Rust naming conventions (snake_case for functions/methods)
  - Match wxWidgets naming where sensible, but adapt to Rust idioms
  - Document any divergence from wxWidgets API

### 4. Code Quality Standards

**Priority: High**

- **Formatting & Linting**:
  - Code must pass `cargo fmt --check` (enforced in CI)
  - Code must pass `cargo clippy --all-targets -- -D warnings` (enforced in CI)
  - No new clippy warnings should be introduced

- **Documentation**:
  - Public APIs must have doc comments (`///`)
  - Examples should be provided for non-trivial functionality
  - Document platform-specific behavior or limitations
  - Include code examples in docstrings where appropriate

- **Error Handling**:
  - Prefer `Result<T, E>` over panicking for recoverable errors
  - Use proper error types (avoid `String` errors)
  - Document when functions can panic

### 5. Testing Requirements

**Priority: High**

- **Build Verification**:
  - Default features must build: `cargo build`
  - All features must build: `cargo build --features="aui,xrc,richtext,stc,media-ctrl,webview"`
  - Examples must build (especially feature-gated ones like `simple_xrc_test`, `simple_stc_test`)

- **Manual Testing**:
  - Visual widgets should be manually tested on at least one platform
  - Verify UI appears correct and events fire as expected
  - Check for visual glitches, layout issues, or crashes

- **Feature Flags**:
  - Optional features (`aui`, `media-ctrl`, `webview`, `stc`, `xrc`, `richtext`) should be properly gated
  - Verify features don't leak into default build

### 6. wxdragon-sys Changes

**Priority: Critical**

Changes to the low-level `wxdragon-sys` crate require extra scrutiny:

- **Bindings Generation**:
  - Verify generated bindings are correct
  - Check that C++ name mangling is handled properly
  - Ensure no undefined symbols at link time

- **Build Script Changes** (`build.rs`):
  - Verify prebuilt library download logic works for all platforms
  - Check that linking flags are correct for each target
  - Test both online (prebuilt) and offline (system wxWidgets) builds if applicable

- **C Wrapper Changes**:
  - C wrapper code should be minimal and safe
  - Verify proper error handling at C boundary
  - Check for potential UB in C code

### 7. Performance Considerations

**Priority: Medium**


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AllenDang/wxDragon](https://github.com/AllenDang/wxDragon) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
