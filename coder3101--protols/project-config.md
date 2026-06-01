---
trigger: always_on
description: Protols is an open-source Language Server Protocol (LSP) implementation for Protocol Buffers (proto) files, written in Rust. It provides intelligent code assistance for protobuf development, including auto-completion, diagnostics, formatting, go-to-definition, hover information, and more.
---

# Protols - Protocol Buffers Language Server

Protols is an open-source Language Server Protocol (LSP) implementation for Protocol Buffers (proto) files, written in Rust. It provides intelligent code assistance for protobuf development, including auto-completion, diagnostics, formatting, go-to-definition, hover information, and more.

Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.

## Working Effectively

### Bootstrap and Build
- Install dependencies and build the project:
  - Rust toolchain is already available (cargo 1.89.0, rustc 1.89.0)
  - Install protoc: `sudo apt update && sudo apt install -y protobuf-compiler` -- takes 2-3 minutes. NEVER CANCEL. Installs libprotoc 3.21.12.
  - clang-format is already installed and available at `/usr/bin/clang-format` (Ubuntu clang-format version 18.1.3)
  - `cargo build --verbose` -- takes about 1 minute to complete. NEVER CANCEL. Set timeout to 90+ minutes for safety.
  - `cargo test --verbose` -- takes about 6 seconds, runs 22 tests. NEVER CANCEL. Set timeout to 30+ minutes.

### Essential Commands
- Check code formatting: `cargo fmt --check` -- takes under 1 second
- Run linter: `cargo clippy` -- takes about 15 seconds. NEVER CANCEL. Set timeout to 30+ minutes.
- Run the binary: `./target/debug/protols --help` or `./target/debug/protols --version`
- Build release version: `cargo build --release` -- takes about 1 minute. NEVER CANCEL. Set timeout to 90+ minutes.
- Test specific functionality: `cargo test <test_name>` for individual tests

### External Dependencies Verification
- **protoc (Protocol Buffers Compiler)**: Required for advanced diagnostics. Install with `sudo apt install -y protobuf-compiler`. Verify with `protoc --version`.
- **clang-format**: Required for code formatting. Already available. Verify with `clang-format --version`.

## Validation and Testing

### Manual Validation Scenarios
After making changes to the LSP functionality, ALWAYS test these scenarios:

1. **Basic Build and Test Validation**:
   - `cargo build` -- should complete in ~1 minute without errors
   - `cargo test --verbose` -- should pass all 22 tests in ~6 seconds  
   - `cargo fmt --check` -- should pass formatting check
   - `cargo clippy` -- should pass linting with no warnings
   - `./target/debug/protols --help` -- should show help message
   - `./target/debug/protols --version` -- should show version 0.12.8

2. **External Dependencies Validation**:
   - `protoc --version` -- should show libprotoc 3.21.12
   - `clang-format --version` -- should show Ubuntu clang-format version 18.1.3
   - Test protoc with sample file: `protoc sample/simple.proto --descriptor_set_out=/tmp/test.desc`
   - Test clang-format with sample file: `clang-format sample/simple.proto`

3. **LSP Functionality Testing**:
   - Test specific LSP features: `cargo test parser::hover::test::test_hover`
   - Test workspace functionality: `cargo test workspace`
   - Test with include paths: `./target/debug/protols --include-paths=/tmp,/home` (will start LSP server)
   - Verify LSP server starts correctly (shows logging directory and waits for input)

4. **Sample File Validation**:
   - Ensure sample proto files in `/sample/` directory are valid
   - Test parsing with `sample/simple.proto`, `sample/everything.proto`, `sample/test.proto`
   - Verify protoc can process sample files without errors

### CRITICAL Build and Test Timing
- **NEVER CANCEL builds or tests** - they may take longer than expected
- **cargo build**: 1 minute typical, set timeout to 90+ minutes
- **cargo test**: 6 seconds typical, set timeout to 30+ minutes  
- **cargo clippy**: 15 seconds typical, set timeout to 30+ minutes
- **External dependency installation**: 2-3 minutes, set timeout to 30+ minutes

### CI Validation Requirements
Always run these commands before committing changes:
- `cargo fmt --check` -- validates code formatting
- `cargo clippy` -- validates code quality and catches common issues
- `cargo test --verbose` -- runs full test suite
- `cargo build --release` -- ensures release build works

## Key Project Structure

### Root Directory
```
├── Cargo.toml              # Main project configuration
├── Cargo.lock              # Dependency lock file
├── README.md               # Project documentation
├── protols.toml            # LSP configuration file
├── .clang-format           # Formatting configuration for proto files
├── src/                    # Main source code
├── sample/                 # Sample proto files for testing
└── .github/workflows/      # CI/CD pipelines
```

### Important Source Files
- `src/main.rs` - Entry point, command-line argument parsing, LSP server setup
- `src/server.rs` - Core LSP server implementation
- `src/lsp.rs` - LSP message handling and protocol implementation
- `src/parser/` - Tree-sitter based proto file parsing
- `src/formatter/` - Code formatting using clang-format
- `src/workspace/` - Workspace and multi-file support

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [coder3101/protols](https://github.com/coder3101/protols) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->
