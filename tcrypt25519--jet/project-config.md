---
trigger: always_on
description: - **Jet** is an LLVM-based JIT compiler for the Ethereum Virtual Machine (EVM)
---

# Jet – Copilot Onboarding Instructions

## What this repo does
- **Jet** is an LLVM-based JIT compiler for the Ethereum Virtual Machine (EVM)
- Rust workspace that JIT-compiles EVM bytecode to LLVM IR, then to native code via LLVM ORC JIT (`inkwell` bindings)
- Ideal for MEV use cases: same contract executed thousands of times with warm data
- Core value: native code execution vs. interpretation, with LLVM optimization passes

## Repository structure

### Workspace layout
- **Root files:**
  - `Cargo.toml` (workspace manifest with 4 members)
  - `Makefile` (build automation, exports LLVM env vars)
  - `.cargo/config.toml` (Android/Termux linker flags)
  - `rust-toolchain.toml` (pins stable Rust channel)
  - `README.md`, `DEVELOPMENT.md` (getting started, dev workflow)
  - `.nextest.toml` (cargo-nextest test runner config)

- **Workspace members (4 crates):**
  1. `crates/jet` - Main compiler crate
     - Edition 2024
     - Parses EVM opcodes, builds LLVM IR, drives JIT
     - Key modules: `builder/{contract,env,manager,ops}.rs`, `engine/mod.rs`, `instructions.rs`
     - Binary: `bin/jetdbg.rs` (debugger/executor)
     - Tests: `tests/test_roms.rs` (EVM opcode integration tests)
  
  2. `crates/jet_runtime` - Runtime execution context
     - Edition 2024
     - Crate type: `["dylib", "lib"]`
     - Builtins invoked from generated LLVM IR
     - Key modules: `{lib,exec,builtins,symbols}.rs`, `binding/mod.rs`
  
  3. `crates/jet_ir` - Shared LLVM IR types
     - Edition 2024
     - Function registry for Jet EVM JIT compiler
  
  4. `crates/jet_push_macros` - Procedural macros
     - Edition 2024
     - Proc-macro crate for test utilities

- **Scripts:** `scripts/{detect-llvm,detect-platform,install-llvm,llvm}.sh`
- **Documentation:** `docs/` (see [Key Documentation](#key-documentation) section)
- **CI:** `.github/workflows/ci.yml` (GitHub Actions, see [CI/CD](#cicd) section)

### Key source files
- `crates/jet/src/`:
  - `lib.rs` - Public API exports
  - `instructions.rs` - EVM instruction enum and opcode mappings
  - `builder/contract.rs` - Bytecode chunking, CFG construction, jump tables
  - `builder/ops.rs` - LLVM IR generation for each EVM opcode
  - `builder/env.rs` - Compilation environment (types, symbols, modes)
  - `builder/manager.rs` - Build orchestration
  - `engine/mod.rs` - LLVM ORC JIT engine, symbol resolution
  - `bin/jetdbg.rs` - CLI debugger for EVM contracts

- `crates/jet_runtime/src/`:
  - `builtins.rs` - Complex operations implemented in Rust (EXP, KECCAK256, etc.)
  - `exec.rs` - Execution context (stack, memory, storage, logs)
  - `symbols.rs` - Runtime symbol name constants
  - `runtime_builder.rs` - LLVM IR declarations for runtime functions

## Toolchain & dependencies

### Rust
- **Version:** Stable (pinned in `rust-toolchain.toml`)
- **Editions:** 2024 (jet, jet_runtime, jet_ir, jet_push_macros)
- **Components:** rustfmt, clippy (for CI)
- **No nightly required** - all operations use stable

### LLVM
- **Version:** 21 (REQUIRED)
- **Detection:** `scripts/detect-llvm.sh` finds `/usr/lib/llvm-21`
- **Environment:** Makefile exports `LLVM_SYS_211_PREFIX` automatically
- **Bindings:** 
  - `inkwell` - Git dependency with `llvm21-1-prefer-dynamic` feature
  - `llvm-sys` - Expects `llvm-config-21` or `LLVM_SYS_211_PREFIX`

### Dependencies
- **LLVM tooling:** `inkwell` (Git: TheDan64/inkwell, branch with LLVM 21 support)
- **Crypto:** `sha3`, `bnum` (256-bit arithmetic), `hex`
- **CLI:** `clap` (derive), `colored`, `syntect` (syntax highlighting)
- **Serialization:** `serde`, `serde_json`, `bincode`
- **Logging:** `log`, `simple_logger`
- **Error handling:** `thiserror`

### Platform support
- **Supported:** Linux (Debian/Ubuntu), macOS (darwin), Android (Termux)
- **Detection:** `scripts/detect-platform.sh` branches by platform
- **Termux-specific:** 
  - `.cargo/config.toml` contains linker flags
  - `CARGO_TARGET_DIR` relocated to `/data/data/com.termux/files/home/.cargo/jet-target`

## Build, test, lint workflow

### CRITICAL: Bootstrap LLVM first
**ALWAYS run this before any cargo command:**
```bash
make install-llvm
```

This:
- Detects platform via `scripts/detect-platform.sh`
- Installs LLVM 21 via apt (Debian/Ubuntu) or appropriate method
- May require `sudo` for system package installation
- Takes 1-3 minutes depending on network/disk speed

**Failure symptoms if LLVM missing:**
- `cargo check` fails with: `No suitable version of LLVM was found system-wide or pointed to by LLVM_SYS_211_PREFIX`
- `llvm-sys` build script cannot find `llvm-config-21`

**Verification:**
```bash
bash scripts/detect-llvm.sh  # Should output: /usr/lib/llvm-21
which llvm-config-21         # Should find executable
```

### Build commands
```bash
# Standard workflow
make check      # cargo check --all-targets --all-features (fastest validation)
make build      # cargo build
make fmt        # cargo fmt --all (auto-format)
make clippy     # cargo clippy --all-targets --all-features -- -D warnings (strict)

# Combined pre-push check
make commit-check  # Runs: fmt-check, check, clippy, test-all
make ci            # Alias for commit-check (mirrors CI pipeline)
```

**Note:** `make commit-check` is the **recommended pre-push command** - it runs all CI checks locally.

### Testing


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tcrypt25519/jet](https://github.com/tcrypt25519/jet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-04 -->
