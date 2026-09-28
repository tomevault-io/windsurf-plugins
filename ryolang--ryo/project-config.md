---
trigger: always_on
description: **Ryo** is a pre-alpha statically-typed, compiled (AOT/JIT) programming language implemented in Rust. See README.md for language philosophy and design goals.
---

# Critical

Be brief.

# Ryo Programming Language - Repository Conventions

**Ryo** is a pre-alpha statically-typed, compiled (AOT/JIT) programming language implemented in Rust. See README.md for language philosophy and design goals.

## Tech Stack & Layout

**Stack:** Rust compiler with Cranelift backend, Zig linker, Logos lexer, Chumsky parser.
**Layout:** Cargo workspace whose members (per the root `Cargo.toml`) are:
- `ryo/` (CLI binary crate)
- `ryo-core/` (shared models: AST, IRs (UIR, TIR), types, diagnostics, and errors)
- `ryo-frontend/` (lexer, indent preprocessor, parser, AST lowering, semantic analysis, builtins, ownership analysis)
- `ryo-backend/` (Cranelift code generation, Zig linking, toolchain management, runtime extraction)
- `ryo-driver/` (pipeline compilation orchestration driver)
- `runtime/` (Ryo static runtime library)
- `build-support/` (shared build-script helpers, e.g. the ryo-runtime archive build used by `ryo/build.rs` and `ryo-backend/build.rs`)

Non-member repository areas (not part of the Cargo workspace):
- `docs/` (spec, roadmap, examples)
- `.github/` (CI)

---

## File Naming Conventions

- **Ryo files:** lowercase with underscores (`error_handling.ryo`, `hello_world.ryo`)
- **Docs:** lowercase with underscores (`getting_started.md`). Special files uppercase (`README.md`, `NOTES.md`, `TODO.md`)
- **Rust files:** lowercase with underscores (`main.rs`, `ast.rs`) following Rust conventions

---

## Critical Syntax Rules

**⚠️ CRITICAL: Python-Style Syntax is MANDATORY**

All Ryo code examples **must** use Python-style colons and indentation, **NOT** curly braces. Braces appear ONLY in f-strings and struct literals (`Point{x=5, y=9}`) — never for blocks.

**Tab Indentation:** Use TABS (not spaces). Mixing tabs/spaces is a compile-time error. One tab = one indentation level.

---

## Documentation Standards

**Code examples:** Use fenced code blocks with language tag (`ryo`).
**Cross-references:** Use relative paths (`[spec](docs/specification.md)`).
**Milestone completion:** When a milestone ships, update `landing/reference/index.html` if the language surface changed (types, literals, builtins, diagnostics) — and remove any "planned" callout the milestone fulfills.
**Committed artifacts are self-contained:** anything under version control (`ISSUES.md`, `benchmarks/README.md` files, code comments, commit messages) must be understandable on its own. Never reference vocabulary or context that lives only in uncommitted scratch — e.g. `docs/superpowers/` specs and plans are gitignored, so committed files must not cite their "Phase 0/1/2" naming or section numbers. Cite the concept inline instead ("the value-range guard-elision work"), optionally with a commit hash or issue ID as a historical pointer.

---

## Build & Test Commands

Standard cargo commands work fully out-of-the-box (even on a clean checkout) because `build.rs` automatically compiles the `ryo-runtime` static library in a separate target directory if it isn't found.

```bash
cargo build                      # Automatically builds the runtime (if missing) and then compiles the compiler
cargo check                      # Check compiler for errors
cargo test                       # Run all unit + integration tests
./scripts/run_linux_tests.sh             # Build Docker image and run entire test suite in Linux (ASan + Valgrind leak detection)
./scripts/check_cranelift.sh [version]   # Diff Ryo's Cranelift version (from Cargo.lock) against another (default: latest) — see below
./scripts/check_file_length.sh           # Fail on Rust files over 2000 lines (no allowlist; CI runs the same check)
cargo run -- run <file>          # JIT compile and execute
cargo run -- build <file>        # AOT compile to binary
cargo run -- toolchain install   # Download Zig linker
cargo run -- toolchain status    # Check Zig status
RUSTFLAGS=-Dwarnings cargo clippy --workspace --all-targets  # Lint; RUSTFLAGS=-Dwarnings matches CI exactly (ci.yml sets it env-wide). `--workspace` is required — bare `--all-targets` only checks the default member `ryo` and misses other crates' test/bench targets
cargo fmt --check                # Check code formatting style
```

**Tracking Cranelift changes.** Ryo is built on the Cranelift backend, so upstream changes can affect codegen. `./scripts/check_cranelift.sh` resolves Ryo's Cranelift version from `Cargo.lock`, queries crates.io and the GitHub API for the exact commit SHAs, and prints the history of commits touching Cranelift's `cranelift/` directory between that version and a target version (default: latest release), handling parallel release-branch history. Pass a version argument to diff against a specific release instead of latest. Use it before bumping the Cranelift dependency to review what changed.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ryolang/ryo](https://github.com/ryolang/ryo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-26 -->
