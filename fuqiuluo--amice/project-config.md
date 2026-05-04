---
trigger: always_on
description: Amice is an LLVM plugin written in Rust that provides code obfuscation transformations for C/C++ programs. It operates as a Clang plugin during compilation, offering multiple obfuscation techniques including string encryption, indirect calls/branches, block shuffling, basic block splitting, and VM flattening.
---

# Copilot Instructions for Amice

## Repository Overview

Amice is an LLVM plugin written in Rust that provides code obfuscation transformations for C/C++ programs. It operates as a Clang plugin during compilation, offering multiple obfuscation techniques including string encryption, indirect calls/branches, block shuffling, basic block splitting, and VM flattening.

**Key Facts:**
- Primary language: Rust (edition 2024)
- Target: LLVM plugin (libamice.so) for use with clang
- Multi-crate workspace: main `amice` crate + `amice-llvm` sub-crate
- Configuration-driven with environment variable overrides
- Supports LLVM versions 11.0 through 20.1 via feature flags

## Critical Build Requirements

**ALWAYS set these environment variables before any Rust commands:**
```bash
export LLVM_SYS_181_PREFIX=/usr/lib/llvm-18
```

**ALWAYS use these cargo flags for the current environment:**
```bash
--no-default-features --features llvm18-1
```

The default feature is `llvm20-1` but the CI environment only has LLVM 18 available. Using wrong LLVM version will cause immediate compilation failures.

## Build Instructions

### Prerequisites Verification
Check LLVM installation before building:
```bash
# Verify LLVM 18 is available
llvm-config-18 --version  # Should output: 18.1.3
clang --version           # Should show clang 18.x
```

### Build Commands (Required Order)

1. **Check/Validate Code:**
```bash
export LLVM_SYS_181_PREFIX=/usr/lib/llvm-18
cargo check --no-default-features --features llvm18-1
```

2. **Debug Build:**
```bash
export LLVM_SYS_181_PREFIX=/usr/lib/llvm-18  
cargo build --no-default-features --features llvm18-1
```

3. **Release Build (Required for Testing):**
```bash
export LLVM_SYS_181_PREFIX=/usr/lib/llvm-18
cargo build --release --no-default-features --features llvm18-1
```

4. **Format Code:**
```bash
cargo fmt
```

5. **Format Check:**
```bash
cargo fmt --check
```

### Build Timing
- Initial build: ~30-60 seconds (includes dependency compilation)
- Incremental builds: ~5-15 seconds
- Release build: ~20-40 seconds
- Clean build: ~60-120 seconds

### Testing

**Unit Tests:**
```bash
export LLVM_SYS_181_PREFIX=/usr/lib/llvm-18
cargo test --no-default-features --features llvm18-1 --lib
```

**Integration Tests (Require Release Build):**
```bash
# First ensure release build exists
export LLVM_SYS_181_PREFIX=/usr/lib/llvm-18
cargo build --release --no-default-features --features llvm18-1

# Then run integration tests
cargo test --no-default-features --features llvm18-1 --test const_strings
```

**Manual Plugin Testing:**
```bash
# After release build, test plugin functionality
clang -fpass-plugin=target/release/libamice.so tests/test1.c -o target/test1
./target/test1  # Should run successfully
```

## Project Architecture

### Directory Structure
```
├── .github/workflows/          # CI/CD workflows
│   ├── rustfmt.yml            # Auto-formatting on PR/push
│   └── generate-structure.yml  # Project structure docs
├── src/                       # Main plugin source
│   ├── aotu/                  # Obfuscation techniques
│   │   ├── string_encryption/ # String obfuscation algorithms
│   │   ├── indirect_call/     # Indirect call obfuscation
│   │   ├── indirect_branch/   # Indirect branch obfuscation  
│   │   ├── shuffle_blocks/    # Basic block reordering
│   │   ├── split_basic_block/ # Block splitting
│   │   └── vm_flatten/        # Virtual machine obfuscation
│   ├── config/                # Configuration handling
│   ├── llvm_utils/           # LLVM utility functions
│   └── lib.rs                # Plugin entry point
├── amice-llvm/               # LLVM FFI bindings
│   ├── src/                  # Rust FFI code
│   ├── cpp/                  # C++ implementation
│   └── build.rs              # Complex LLVM build logic
├── tests/                    # Integration tests
│   ├── *.c                   # C test programs
│   └── *.rs                  # Rust test drivers
├── Cargo.toml               # Main project config
└── .rustfmt.toml            # Code formatting rules
```

### Configuration System
- **Environment Variables:** `AMICE_*` variables control runtime behavior
- **Config Files:** Support TOML/YAML/JSON via `AMICE_CONFIG_PATH` 
- **Overlay System:** Environment variables override config file settings

### Key Environment Variables
- `AMICE_STRING_ALGORITHM`: `xor` | `simd_xor`
- `AMICE_STRING_DECRYPT_TIMING`: `lazy` | `global`
- `AMICE_STRING_STACK_ALLOC`: `true` | `false`
- `RUST_LOG`: Set to `debug` for detailed plugin logs

## Validation Pipeline

### Automated Checks (GitHub Actions)
1. **rustfmt**: Auto-formats code on push/PR to main/master
2. **Project Structure**: Manual workflow to update PROJECT_STRUCTURE.md

### Manual Validation Steps
1. **Build Verification:**
   ```bash
   export LLVM_SYS_181_PREFIX=/usr/lib/llvm-18
   cargo build --release --no-default-features --features llvm18-1
   ```

2. **Plugin Functionality Test:**
   ```bash
   clang -fpass-plugin=target/release/libamice.so tests/test1.c -o target/test1

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fuqiuluo/amice](https://github.com/fuqiuluo/amice) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
