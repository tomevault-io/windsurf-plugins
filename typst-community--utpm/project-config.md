---
trigger: always_on
description: **UTPM** (Unofficial Typst Package Manager) is a command-line package manager written in Rust for managing [Typst](https://typst.app/) packages. It provides comprehensive tools for creating, managing, linking, and publishing Typst packages for both local development and remote distribution via Typst Universe.
---

# UTPM - Copilot Instructions

## Project Overview

**UTPM** (Unofficial Typst Package Manager) is a command-line package manager written in Rust for managing [Typst](https://typst.app/) packages. It provides comprehensive tools for creating, managing, linking, and publishing Typst packages for both local development and remote distribution via Typst Universe.

### Key Information
- **Language**: Rust (Edition 2024)
- **Version**: 0.3.0
- **License**: MIT
- **Repository**: https://github.com/typst-community/utpm
- **Primary Dependencies**: 
  - `clap` (4.5.39) - CLI parsing with derive macros
  - `typst-kit` (0.13.1) - Core Typst functionality and package downloading
  - `typst-syntax` (0.13.1) - Parsing and working with Typst manifests
  - `tokio` (1.45.1) - Async runtime with full features
  - `ignore` (0.4.23) - File filtering with gitignore-style patterns
  - `octocrab` (0.44.1) - GitHub API client
  - `tracing` (0.1.41) - Structured logging and instrumentation
  - `inquire` (0.7.5) - Interactive CLI prompts
  - `semver` (1.0.26) - Semantic versioning
  - `serde` (1.0) - Serialization/deserialization
  - `shadow-rs` (1.1.1) - Build-time information
  - `ecow` (0.2) - Efficient copy-on-write data structures

### Project Philosophy
- **Dry-run first**: All destructive operations support `--dry-run` mode
- **Flexible output**: Support for JSON, YAML, TOML, HJSON, and text output formats
- **Respectful of conventions**: Honors `.gitignore`, `.typstignore`, and custom ignore files
- **Developer-friendly**: Clear error messages, verbose logging, and intuitive CLI
- **Code quality**: Enforced formatting (rustfmt) and linting (Clippy) standards

### Recent Changes & Features

**Version 0.3.0 Updates:**

1. **Metadata Command** (Issue #89)
   - New command: `utpm prj metadata` to extract metadata from `typst.toml`
   - Supports extracting specific fields with `-f` flag (name, version, authors, etc.)
   - Outputs in multiple formats (text, JSON, YAML, TOML, HJSON)
   - Ideal for shell scripts and CI/CD pipelines
   - Properly converts `EcoString` types to `String` for serialization

2. **File Exclusion Documentation** (Issue #88)
   - Documented `[tool.utpm]` exclude patterns in README
   - Added comprehensive examples in `assets/typst.toml.example`
   - Fixed bug where `typst_ignore` flag was not respected in `link` and `publish` commands

3. **Code Quality Standards**
   - Added `rustfmt.toml` for consistent code formatting (stable features only)
   - Added `clippy.toml` for Clippy linting configuration
   - Added `.editorconfig` for cross-editor consistency
   - Created `justfile` with common development commands
   - Git hooks support via `just setup-hooks`
   - CI already includes format and lint checks

4. **Documentation Reorganization**
   - Created `docs/` directory for all developer documentation
   - Added `docs/GUIDE.md` - Comprehensive guide for users, package authors, and contributors
   - Moved `docs/CONTRIBUTING.md` - Contribution guidelines and code standards
   - Moved `docs/DEVELOPMENT.md` - Development workflow and tooling
   - Added `docs/TESTING.md` - Comprehensive testing guide and documentation
   - `.github/copilot-instructions.md` remains in .github/ for AI assistant context
   - All documentation accessible and beginner-friendly

5. **Comprehensive Test Suite** (November 2025)
   - Created full test infrastructure with 60+ tests
   - Test structure: `tests/common/mod.rs` with 10 helper functions
   - Unit tests: `tests/utils_tests.rs` - 18 tests for utility functions
   - Command tests: `tests/command_tests.rs` - 24 tests for all commands
   - Integration tests: `tests/integration_tests.rs` - 14 end-to-end workflow tests
   - Test dependencies: `tempfile = "3.15"` for isolated testing
   - Library exports: `src/lib.rs` exposes modules for testing
   - Enhanced `justfile` with test commands (test-unit, test-integration, test-module, test-coverage, test-watch)
   - Documentation: `docs/TESTING.md` with comprehensive testing guide
   - All tests passing, environment isolation working correctly

6. **Complete API Documentation** (November 2025)
   - All 60+ public functions now have comprehensive documentation
   - Documented all utility functions in `src/utils.rs` (copy_dir_all, try_find_path, try_find, write_manifest)
   - Documented all path functions in `src/utils/paths.rs` (get_data_dir, get_cache_dir, c_packages, d_packages, etc.)
   - Documented all git operations in `src/utils/git.rs` (exist_git, clone_git, push_git, pull_git, add_git, commit_git)
   - Documented all command run functions and helpers
   - Added function descriptions, parameter explanations, return values, and error conditions
   - Documentation includes usage examples and edge cases
   - cargo doc builds successfully without warnings

7. **Feature Flag Cleanup** (November 2025)
   - Removed all command-specific feature flags (install, clone, publish, etc.)
   - Kept only output format features: `output_json`, `output_yaml`, `output_hjson`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [typst-community/utpm](https://github.com/typst-community/utpm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
