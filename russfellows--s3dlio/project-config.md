---
trigger: always_on
description: **Prime Directive**: If told to investigate something, I will ONLY investigate, and then report and possibly document my findings. I will NEVER assume to proceed to making ANY code changes, unless SPECIFICALLY asked to do so.
---

# s3dlio AI Coding Agent Instructions

---

## 🚨 Prime Directive

**Prime Directive**: If told to investigate something, I will ONLY investigate, and then report and possibly document my findings. I will NEVER assume to proceed to making ANY code changes, unless SPECIFICALLY asked to do so.

**Secondary Directive**: Whenever given a task, first check that I am not violating my Prime Directive before proceeding.

---

## Project Overview
s3dlio is a high-performance, multi-protocol storage library built in Rust with Python bindings, designed for AI/ML workloads. It provides universal copy operations across S3, Azure, local file systems, and DirectIO with near line-speed performance.

**Current Version**: v0.9.5 (October 2025)

### Performance Targets
- **Read (GET)**: Minimum 5 GB/s (50 Gb/s) sustained, target higher
- **Write (PUT)**: Minimum 2.5 GB/s (25 Gb/s) sustained, target higher
- **Infrastructure**: Tested against Vast storage systems with bonded 100 Gb ports
- **S3 Compatibility**: MinIO, Vast, AWS S3, and other S3-compatible storage systems

## Core Architecture

### Dual-Backend System
The project uses **mutually exclusive backend selection** at compile-time:
- `native-backends` feature: AWS SDK + Azure SDK (**DEFAULT** - recommended for production)
- `arrow-backend` feature: Apache Arrow object_store implementation (experimental, optional)

```bash
# Default build (uses native-backends)
cargo build --release

# Explicit native-backends (RECOMMENDED)
cargo build --no-default-features --features native-backends

# Experimental arrow backend (NOT RECOMMENDED for production)
cargo build --no-default-features --features arrow-backend
```

**Critical**: These features are mutually exclusive by design (`compile_error!` in `src/lib.rs`).

**Backend Status**:
- **native-backends**: Default, proven performance (5+ GB/s reads, 2.5+ GB/s writes), production-ready
- **arrow-backend**: Experimental only, no proven performance benefit, kept for comparison testing

### Build Quality Standards

**CRITICAL: Build Timing — NEVER Interrupt a Release Build**

`cargo build --release` takes **~10 minutes** on this machine. This is normal and expected.

- **NEVER** interrupt a running `cargo build --release` with Ctrl-C or by starting a new command
- **NEVER** grow impatient and kill the build partway through
- **NEVER** assume the build is hung just because it is quiet — Rust release builds are silent during compilation
- After starting a release build, **wait the full ~10 minutes** for it to complete before doing anything else
- If you need a faster feedback loop, use a **debug build** instead: `cargo build` (no `--release`)
  - Debug builds finish in ~1-2 minutes and catch all errors and warnings
  - Only use `--release` when you need the final optimized binary for testing

**Build command cheat-sheet**:
```bash
# Fast iteration / error checking (~1-2 min)
cargo build

# Final binary for performance testing (~10 min, DO NOT INTERRUPT)
cargo build --release

# Check for errors without producing binary (fastest, ~30 sec)
cargo check

# Lint
cargo clippy
```

**CRITICAL: Zero Warnings Policy**
- ALL builds MUST be warning-free before commits
- Never use quick fixes like `_` prefix to silence unused variable warnings
- Unused variables often indicate logic errors that must be investigated
- Unused imports must be removed, not ignored

**Pre-Commit Checklist**:
1. Run `cargo build --release` and verify ZERO warnings
2. Run `cargo clippy` and fix all issues
3. Investigate root cause of any warning - do not suppress without understanding
4. If unsure about a warning, ask for clarification before committing

**Shell Command Best Practices**:
- Never use exclamation marks (`!`) in Python print statements or shell commands
- Exclamation marks cause shell escaping issues in bash
- Use simple declarative messages instead: "Import successful" not "Import successful!"

**Warning Investigation Process**:
```bash
# Check for warnings
cargo build --release 2>&1 | grep -i warning

# Get full details
cargo build --release 2>&1 | grep -A 10 warning

# For clippy suggestions
cargo clippy --all-targets --all-features
```

**Common Warning Anti-Patterns** (DO NOT DO):
- ❌ Adding `_` prefix to silence unused variable warnings
- ❌ Using `#[allow(unused)]` without understanding why
- ❌ Importing modules "just in case" they might be needed
- ❌ Leaving debug code that uses variables only in certain configs

**Correct Approach**:
- ✅ Remove unused imports completely
- ✅ Investigate why variables aren't used (logic bug?)
- ✅ Use feature gates if code is conditionally compiled
- ✅ Refactor to eliminate the warning's root cause

### Dependency Management

**aws-smithy-http-client Patches: REMOVED**
- Custom patches in `fork-patches/aws-smithy-http-client/` are NOT used by default
- Patches showed no measurable performance benefit
- Removed from `[patch.crates-io]` to avoid forcing downstream users to patch
- Fork preserved for reference/experimentation but not required for builds

### Public API Structure

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [russfellows/s3dlio](https://github.com/russfellows/s3dlio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-18 -->
