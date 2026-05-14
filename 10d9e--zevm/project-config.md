---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

ZEVM is a high-performance Ethereum Virtual Machine (EVM) implementation written in Zig. It is a port of the Rust [revm](https://github.com/bluealloy/revm) implementation, providing:

1. **Complete EVM Implementation**: Full support for Ethereum protocol up to Prague hardfork
2. **Production-Ready Precompiles**: All 18 standard Ethereum precompiled contracts implemented
3. **Cross-Platform Support**: Works on macOS, Linux, and Windows with static linking
4. **Type-Safe Design**: Leverages Zig's compile-time guarantees for safety and performance

The project maintains **100% feature parity** with the Rust revm reference implementation, verified through comprehensive testing and comparison.

## Build and Development Commands

### Essential Commands

```bash
# Build using Makefile (recommended - auto-detects OS and installs deps)
make
make build          # Build only
make install-deps   # Install dependencies only
make test           # Run all tests
make clean          # Clean build artifacts

# Manual Zig build commands
zig build                    # Build all targets
zig build test              # Run tests
zig build -Dblst=false      # Build without blst library
zig build -Dmcl=false       # Build without mcl library

# Build specific examples
zig build precompile_example
zig build zevm-test
zig build zevm-bench
```

### Platform-Specific Notes

- **macOS**: Uses Homebrew for dependencies (`/opt/homebrew/include`)
- **Linux**: Uses apt-get/dnf for dependencies (`/usr/local/include`)
- **Windows**: Uses vcpkg (see CROSS_PLATFORM.md)

The Makefile automatically detects the OS and installs dependencies accordingly.

## Architecture

The project follows a modular architecture mirroring revm's structure:

### Core Modules

- **`src/primitives/`**: Core types (U256, Address, Hash), constants, and hardfork definitions
- **`src/bytecode/`**: Opcode definitions, bytecode analysis, and EIP-7702 support
- **`src/state/`**: Account info, storage, and state transitions
- **`src/database/`**: Pluggable database interface with in-memory implementation
- **`src/context/`**: Block, transaction, and configuration management
- **`src/interpreter/`**: Stack-based EVM interpreter with gas tracking
- **`src/precompile/`**: All 18 Ethereum precompiled contracts
- **`src/handler/`**: Transaction execution orchestration
- **`src/inspector/`**: Debugging and profiling tools

### Key Design Patterns

1. **Modular Structure**: Each module is self-contained with a `main.zig` entry point
2. **Static Linking**: All crypto libraries (`blst`, `mcl`) are statically linked for self-contained binaries
3. **Spec-Based Execution**: Hardfork support via `SpecId` enum, with precompiles adapting gas costs per spec
4. **Error Handling**: Uses Zig's error union types (`PrecompileError`, `InstructionResult`, etc.)
5. **Memory Safety**: Leverages Zig's memory management (no GC, explicit allocators)

### Important Interfaces

1. **PrecompileId**: Union enum identifying precompiles, supports Custom variants
2. **PrecompileSpecId**: Enum for hardfork specs (Homestead, Byzantium, Istanbul, Berlin, Cancun, Prague, Osaka)
3. **Database Interface**: Pluggable state storage (see `src/database/main.zig`)
4. **Inspector Interface**: Hooks for transaction tracing and debugging
5. **Context**: Manages execution state, environment, and configuration

## Precompiles

All 18 standard Ethereum precompiles are implemented:

### Core Precompiles
- **Identity** (0x04): Data copy
- **SHA256** (0x02): SHA-256 hash function
- **RIPEMD160** (0x03): RIPEMD-160 hash function
- **ECRECOVER** (0x01): Elliptic curve signature recovery (libsecp256k1)

### Advanced Precompiles
- **ModExp** (0x05): Modular exponentiation (Byzantium/Berlin/Osaka variants)
- **BN254** (0x06-0x08): BN254 curve operations (mcl library)
- **Blake2F** (0x09): Blake2 compression function
- **KZG Point Evaluation** (0x0A): KZG commitment verification (blst library)
- **BLS12-381** (0x0B-0x11): All 7 BLS12-381 precompiles (blst library)
- **P256Verify** (0x100): secp256r1 signature verification (OpenSSL)

### Precompile Features

- **PrecompileId.name()**: Returns EIP-7910 standardized names
- **PrecompileId.precompile()**: Gets spec-appropriate precompile implementation
- **PrecompileId.Custom**: Support for custom precompile identifiers
- **Gas Cost Adaptation**: Precompiles automatically adjust gas costs based on hardfork

## Dependencies

### Required External Libraries

1. **libsecp256k1**: ECRECOVER precompile (typically via package manager)
2. **OpenSSL**: P256Verify precompile (typically via package manager)
3. **blst**: BLS12-381 and KZG operations (built from source or installed)
4. **mcl**: BN254 operations (built from source or installed)

### Dependency Management

- **Makefile**: Automatically installs dependencies via OS-specific package managers
- **build.zig**: Handles linking and include paths for all libraries
- **Static Linking**: All binaries statically link `blst` and `mcl` for portability
- **Fallback**: Libraries can be disabled with `-Dblst=false` or `-Dmcl=false`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [10d9e/zevm](https://github.com/10d9e/zevm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
