---
trigger: always_on
description: This document provides guidelines for working with GitHub Copilot when contributing to the RustPython project.
---

# GitHub Copilot Instructions for RustPython

This document provides guidelines for working with GitHub Copilot when contributing to the RustPython project.

## Project Overview

RustPython is a Python 3 interpreter written in Rust, implementing Python 3.14.0+ compatibility. The project aims to provide:

- A complete Python-3 environment entirely in Rust (not CPython bindings)
- A clean implementation without compatibility hacks
- Cross-platform support, including WebAssembly compilation
- The ability to embed Python scripting in Rust applications

## Repository Structure

- `src/` - Top-level code for the RustPython binary
- `vm/` - The Python virtual machine implementation
  - `builtins/` - Python built-in types and functions
  - `stdlib/` - Essential standard library modules implemented in Rust, required to run the Python core
- `compiler/` - Python compiler components
  - `parser/` - Parser for converting Python source to AST
  - `core/` - Bytecode representation in Rust structures
  - `codegen/` - AST to bytecode compiler
- `Lib/` - CPython's standard library in Python (copied from CPython). **IMPORTANT**: Do not edit this directory directly; The only allowed operation is copying files from CPython.
- `derive/` - Rust macros for RustPython
- `common/` - Common utilities
- `extra_tests/` - Integration tests and snippets
- `stdlib/` - Non-essential Python standard library modules implemented in Rust (useful but not required for core functionality)
- `wasm/` - WebAssembly support
- `jit/` - Experimental JIT compiler implementation
- `pylib/` - Python standard library packaging (do not modify this directory directly - its contents are generated automatically)

## AI Agent Rules

**CRITICAL: Git Operations**
- NEVER create pull requests directly without explicit user permission
- NEVER push commits to remote without explicit user permission
- Always ask the user before performing any git operations that affect the remote repository
- Commits can be created locally when requested, but pushing and PR creation require explicit approval

**CRITICAL: Pre-commit Checks**
- Before creating ANY commit, you MUST run `prek run --all-files` (or `pre-commit run --all-files`) AND the full test suite. Both must pass — do not commit if either fails.
- Test commands are documented in the [Testing](#testing) section below. At minimum run `cargo test --workspace --exclude rustpython_wasm --exclude rustpython-venvlauncher`; if the change touches `extra_tests/snippets/` run `pytest -v` there too, and if it touches `Lib/` or interpreter behavior, run the relevant `cargo run --release -- -m test <module>` modules.
- If a hook auto-fixes files (e.g. `ruff-format`, `rustfmt`), re-stage the fixes, re-run `prek` until it reports a clean pass, then re-run the tests, then commit.
- NEVER bypass these checks with `--no-verify`, `--no-gpg-sign`, or by skipping tests "because the change is small". If a hook or test fails, fix the underlying issue and create a new commit — do not amend or force the failing commit through.

## Important Development Notes

### Running Python Code

When testing Python code, always use RustPython instead of the standard `python` command:

```bash
# Use this instead of python script.py
cargo run -- script.py

# For interactive REPL
cargo run

# With specific features
cargo run --features jit

# Release mode (recommended for better performance)
cargo run --release -- script.py
```

### Comparing with CPython

When you need to compare behavior with CPython or run test suites:

```bash
# Use python command to explicitly run CPython
python my_test_script.py

# Run RustPython
cargo run -- my_test_script.py
```

### Working with the Lib Directory

The `Lib/` directory contains Python standard library files copied from the CPython repository. Important notes:

- These files should be edited very conservatively
- Modifications should be minimal and only to work around RustPython limitations
- Tests in `Lib/test` often use one of the following markers:
  - Add a `# TODO: RUSTPYTHON` comment when modifications are made
  - `unittest.skip("TODO: RustPython <reason>")`
  - `unittest.expectedFailure` with `# TODO: RUSTPYTHON <reason>` comment

#### Choosing the right marker

When marking a test that fails on RustPython, prefer one of the following forms:

```python
@unittest.expectedFailure  # TODO: RUSTPYTHON; <reason>
# or
@unittest.expectedFailureIf(<condition>, "TODO: RUSTPYTHON; <reason>")
```

If the test would crash the interpreter (segfault, Rust panic, abort, infinite loop), use `skip` instead so the rest of the suite can still run:

```python
@unittest.skip("TODO: RUSTPYTHON; <reason>")
# or
@unittest.skipIf(<condition>, "TODO: RUSTPYTHON; <reason>")
```

**When to use which:**

- **Prefer `expectedFailure` / `expectedFailureIf`** by default. The test body still runs, so if RustPython is later fixed, the unexpected pass surfaces immediately and the decorator can be removed. Use the conditional `*If` form when the failure is environment-specific (e.g., a platform or build flag).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [RustPython/RustPython](https://github.com/RustPython/RustPython) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
