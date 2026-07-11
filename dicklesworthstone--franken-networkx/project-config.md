---
trigger: always_on
description: > Guidelines for AI coding agents working in this Rust codebase.
---

# AGENTS.md — FrankenNetworkX

> Guidelines for AI coding agents working in this Rust codebase.

---

## RULE 0 - THE FUNDAMENTAL OVERRIDE PREROGATIVE

If I tell you to do something, even if it goes against what follows below, YOU MUST LISTEN TO ME. I AM IN CHARGE, NOT YOU.

---

## RULE NUMBER 1: NO FILE DELETION

**YOU ARE NEVER ALLOWED TO DELETE A FILE WITHOUT EXPRESS PERMISSION.** Even a new file that you yourself created, such as a test code file. You have a horrible track record of deleting critically important files or otherwise throwing away tons of expensive work. As a result, you have permanently lost any and all rights to determine that a file or folder should be deleted.

**YOU MUST ALWAYS ASK AND RECEIVE CLEAR, WRITTEN PERMISSION BEFORE EVER DELETING A FILE OR FOLDER OF ANY KIND.**

---

## Irreversible Git & Filesystem Actions — DO NOT EVER BREAK GLASS

1. **Absolutely forbidden commands:** `git reset --hard`, `git clean -fd`, `rm -rf`, or any command that can delete or overwrite code/data must never be run unless the user explicitly provides the exact command and states, in the same message, that they understand and want the irreversible consequences.
2. **No guessing:** If there is any uncertainty about what a command might delete or overwrite, stop immediately and ask the user for specific approval. "I think it's safe" is never acceptable.
3. **Safer alternatives first:** When cleanup or rollbacks are needed, request permission to use non-destructive options (`git status`, `git diff`, `git stash`, copying to backups) before ever considering a destructive command.
4. **Mandatory explicit plan:** Even after explicit user authorization, restate the command verbatim, list exactly what will be affected, and wait for a confirmation that your understanding is correct. Only then may you execute it—if anything remains ambiguous, refuse and escalate.
5. **Document the confirmation:** When running any approved destructive command, record (in the session notes / final response) the exact user text that authorized it, the command actually run, and the execution time. If that record is absent, the operation did not happen.

---

## Git Branch: ONLY Use `main`, NEVER `master`

**The default branch is `main`. The `master` branch exists only for legacy URL compatibility.**

- **All work happens on `main`** — commits, PRs, feature branches all merge to `main`
- **Never reference `master` in code or docs** — if you see `master` anywhere, it's a bug that needs fixing
- **The `master` branch must stay synchronized with `main`** — after pushing to `main`, also push to `master`:
  ```bash
  git push origin main:master
  ```

**If you see `master` referenced anywhere:**
1. Update it to `main`
2. Ensure `master` is synchronized: `git push origin main:master`

---

## Toolchain: Rust & Cargo

We only use **Cargo** in this project, NEVER any other package manager.

- **Edition:** Rust 2024 (nightly required — see `rust-toolchain.toml`)
- **Dependency versions:** Explicit versions for stability
- **Configuration:** Cargo.toml workspace with `workspace = true` pattern
- **Unsafe code:** Forbidden by default (`#![forbid(unsafe_code)]`). If narrow unsafe usage is unavoidable, isolate it behind audited interfaces and tests.

### Key Dependencies

| Crate | Purpose |
|-------|---------|
| `serde` + `serde_json` | Serialization |
| `thiserror` | Ergonomic error type derivation |
| `tracing` | Structured logging and diagnostics |

### Release Profile

The release build optimizes for performance (this is a library, not a binary):

```toml
[profile.release]
opt-level = 3       # Maximum performance optimization
lto = true          # Link-time optimization
codegen-units = 1   # Single codegen unit for better optimization
strip = true        # Remove debug symbols
```

### Python Bindings (Maturin + PyO3)

The project includes Python bindings via PyO3, built with Maturin. The native extension module is `franken_networkx._fnx`.

**Prerequisites:**
- Python 3.10+
- `pip install maturin pytest hypothesis networkx`

**Dev loop:**
```bash
# Build and install into current Python environment
maturin develop --features pyo3/abi3-py310

# With release optimizations (recommended for benchmarks)
maturin develop --release --features pyo3/abi3-py310

# Via rch (remote compilation + local install)
rch exec -- maturin develop --features pyo3/abi3-py310
```

**Testing Python bindings:**
```bash
# Run all Python tests
pytest tests/python/ -v --tb=long

# Run specific test files
pytest tests/python/test_conformance.py -v
pytest tests/python/test_error_messages.py -v
pytest tests/python/test_hypothesis.py -v
pytest tests/python/test_thread_safety.py -v

# Skip slow tests
pytest tests/python/ -v -m "not slow"
```

**Building wheels:**
```bash
# Build a wheel for the current platform
maturin build --release

# Build sdist
maturin sdist
```

**Key files:**
| Path | Purpose |
|------|---------|
| `pyproject.toml` | Maturin config, Python project metadata |
| `crates/fnx-python/` | PyO3 binding crate (lib.rs, algorithms.rs, digraph.rs, generators.rs, readwrite.rs, views.rs) |
| `python/franken_networkx/` | Python package (\_\_init\_\_.py, backend.py, \_fnx.pyi) |
| `tests/python/` | Python test suite (conftest.py, test\_\*.py) |


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Dicklesworthstone/franken_networkx](https://github.com/Dicklesworthstone/franken_networkx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-11 -->
