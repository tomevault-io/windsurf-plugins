---
trigger: always_on
description: This document provides instructions for AI agents (Claude, etc.) working on the serde-nixos codebase.
---

# Agent Development Workflow

This document provides instructions for AI agents (Claude, etc.) working on the serde-nixos codebase.

## Project Overview

**serde-nixos** is a Rust procedural macro library that generates NixOS type definitions from Rust structures. It bridges the gap between Rust configuration types and NixOS module system, ensuring type safety across both ecosystems.

**Languages:** Rust, Nix
**Build System:** Cargo (Rust), Nix Flakes
**Testing:** Rust unit/integration tests, NixOS VM tests

## ⚠️ Critical Development Rules

### NEVER Remove Tests or Checks Without Permission

**This is a hard rule. Violations are not acceptable.**

1. **Do NOT remove or disable tests** - Ever. If a test is failing:
   - Fix the code to make the test pass, OR
   - Fix the test if it's incorrectly written, OR
   - Ask the user for guidance if the test represents outdated requirements

2. **Do NOT remove checks from `nix flake check`** - The flake checks exist for a reason:
   - `rust-ci` - Comprehensive Rust check package that runs:
     - Code formatting (cargo fmt --check)
     - Linting (cargo clippy with warnings as errors)
     - All tests (cargo test --all)
     - Workspace build (cargo build --all --release)
     - Examples build (cargo build --examples --release)
   - `nix-fmt` - Ensures Nix code is formatted
   - `test-service-builds` - Ensures integration test service builds
   - `nixos-integration` - Ensures NixOS VM test passes
   - `pre-commit` - Ensures pre-commit hooks work (formatting only)

3. **If checks fail in Nix sandbox** - Don't remove them. Instead:
   - Ask the user about the intended behavior
   - Propose alternative solutions
   - Fix the underlying issue
   - NEVER silently remove checks

4. **When tempted to remove checks** - STOP and ask:
   - "Why is this check failing?"
   - "What is the check protecting against?"
   - "What would be the risk if this check didn't exist?"
   - "Should I ask the user before removing this?"

5. **Always ask first** - If you believe a test or check should be removed:
   - Explain WHY you think it should be removed
   - Describe what the check was doing
   - Explain the implications of removing it
   - Wait for explicit user approval

**Remember:** Tests and checks are documentation of requirements. Removing them without permission is equivalent to changing requirements without approval.

### Keep Configuration Files Minimal

When creating or updating configuration files (.gitattributes, .editorconfig, etc.):

1. **Only include file types that actually exist in the project**
   - Don't add definitions for .json if project has no JSON files
   - Don't add rules for .sh/.bash if project has no shell scripts
   - Keep it minimal and relevant

2. **Check what file types exist first**
   ```bash
   find . -type f -not -path '*/\.*' -not -path '*/target/*' | sed 's/.*\.//' | sort | uniq
   ```

3. **This project uses:**
   - Rust: `.rs`
   - Nix: `.nix`
   - Cargo: `.toml`, `.lock`
   - Documentation: `.md`
   - Licenses: `LICENSE-MIT`, `LICENSE-APACHE`
   - GitHub Actions: `.yml`
   
   Don't add rules for other file types unless they're actually added to the project.

## Code Quality Standards

All code contributions must meet these quality standards:

### 1. Rust Code Quality

#### Formatting
- **Tool:** `rustfmt`
- **Standard:** Rust 2021 edition formatting
- **Command:** `cargo fmt --all`
- **CI Check:** `cargo fmt --all -- --check`

#### Linting
- **Tool:** `clippy`
- **Standard:** All clippy warnings must be addressed
- **Command:** `cargo clippy --all-targets --all-features`
- **Allowed:** Only explicitly documented exceptions in code
- **CI Check:** `cargo clippy --all-targets --all-features -- -D warnings`

#### Code Style
- Use meaningful variable names
- Add doc comments (`///`) for all public items
- Keep functions focused and under 100 lines
- Prefer explicit types over inference in public APIs
- Use `Result` and `Option` properly, avoid `unwrap()` in library code

### 2. Nix Code Quality

#### Formatting
- **Tool:** `nixfmt` or `nixpkgs-fmt`
- **Standard:** Nixpkgs style guide
- **Command:** `nixfmt flake.nix integration-test/*.nix`
- **Consistency:** All Nix files must be formatted

#### Style
- Use `let...in` for local bindings
- Prefer `mkOption` with explicit types
- Document all NixOS options
- Keep expressions readable (break long lines)

### 3. Documentation

All code must be documented:

- **Public APIs:** Full doc comments with examples
- **Modules:** Module-level documentation
- **Complex Logic:** Inline comments explaining "why"
- **Tests:** Test names that describe what's being tested
- **Examples:** Working, tested examples for features

## Testing Requirements

### Unit Tests

**Location:** `serde-nixos/src/`, `serde-nixos-macros/src/`
**Command:** `cargo test --lib`

Requirements:
- Test all public functions
- Test edge cases (empty inputs, invalid data, etc.)
- Use descriptive test names: `test_feature_behavior`
- Include both positive and negative test cases

Example:
```rust
#[test]
fn test_auto_doc_enabled() {
    #[derive(NixosType)]
    #[nixos(auto_doc)]
    struct Config {
        /// Field doc
        field: String,
    }

    let opts = Config::nixos_options();

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [poelzi/serde-nixos](https://github.com/poelzi/serde-nixos) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-01 -->
