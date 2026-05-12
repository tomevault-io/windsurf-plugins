---
trigger: always_on
description: Minimal, correct, and well-tested Ethereum Virtual Machine (EVM) implementation in Zig, prioritizing specification compliance, clarity, and hardfork support (Berlin through Prague).
---

# Guillotine Mini - EVM Implementation

Minimal, correct, and well-tested Ethereum Virtual Machine (EVM) implementation in Zig, prioritizing specification compliance, clarity, and hardfork support (Berlin through Prague).

## Quick Start

```bash
zig build              # Build all modules
zig build test         # Run unit + spec tests
zig build specs        # Run ethereum/tests validation
zig build wasm         # Build WebAssembly library
zig build test-watch   # Interactive test runner

# Debug failing tests (recommended)
bun scripts/isolate-test.ts "test_name"  # Max debug output + analysis
bun scripts/test-subset.ts "pattern"     # Filter test categories
```

## Prerequisites

- Zig 0.15.1+ (core build system)
- Cargo (required for BN254/ARK cryptographic dependencies)
- Python 3.8+ (test generation and reference implementation)
- uv (Python package manager for spec fixture generation): `brew install uv`
- Bun (TS helpers/agents): `brew install bun`

## Architecture

```
src/
├── evm.zig               # Orchestrator: state, storage, gas refunds, nested calls
├── frame.zig             # Bytecode interpreter: stack, memory, PC, per-opcode logic
├── host.zig              # Abstract state backend interface
├── hardfork.zig          # Hardfork detection and feature flags
├── opcode.zig            # Opcode definitions and utilities
├── trace.zig             # EIP-3155 trace generation
└── errors.zig            # Error types

External Dependencies (fetched via zig build):
├── primitives            # Ethereum types (Address, u256, gas constants, RLP, ABI, etc.)
│                         # Source: https://github.com/evmts/primitives
├── crypto                # Cryptographic primitives (keccak256, secp256k1, BLS12-381)
└── precompiles           # Ethereum precompiled contracts
```

### Core Responsibilities

| Component | Responsibility |
|-----------|----------------|
| **Evm** | State management, storage, gas refunds, warm/cold tracking, nested calls |
| **Frame** | Stack, memory, PC, bytecode interpretation, opcode execution |
| **Host** | Pluggable state backend (balances, nonces, code, logs, self-destruct) |
| **Hardfork** | Gas cost adjustments, feature flag guards (`isAtLeast()`, `isBefore()`) |

**Allocation strategy**: Arena allocator for transaction-scoped memory (freed at transaction end).

## Testing and Debugging

### Test Types

| Type | Purpose | Command |
|------|---------|---------|
| Unit tests | Inline `test` blocks | `zig build test` |
| Spec tests | ethereum/tests GeneralStateTests | `zig build specs` |
| Filtered tests | By hardfork/EIP/opcode | `TEST_FILTER="Cancun" zig build specs` |
| Trace tests | EIP-3155 trace capture/comparison | `zig build test-trace` |
| Watch mode | Auto-reload on changes | `zig build test-watch` |
| Engine tests | Consensus-layer format tests | `INCLUDE_ENGINE_TESTS=1 zig build specs` |

### Engine API Tests (Optional)

`blockchain_test_engine` format tests are disabled by default because they test consensus-layer functionality (block validation, Engine API payloads) rather than core EVM execution. To include them:

```bash
INCLUDE_ENGINE_TESTS=1 zig build specs
```

### Test Scope and Filtering

- **Hardfork**: `Cancun`, `Shanghai`, `London`, `Berlin`, `Merge`, `Prague`
- **EIP**: `transientStorage`, `push0`, `MCOPY`, `warmcoinbase`
- **Opcode**: `add`, `mul`, `sstore`, `sload`, `call`, `create2`
- **Category**: `vmArithmeticTest`, `vmBitwiseLogicOperation`, `vmIOandFlowOperations`

### Helper Scripts

<details>
<summary><b>isolate-test.ts - Test Isolation Helper (recommended)</b></summary>

```bash
bun scripts/isolate-test.ts "exact_test_name"
```

**Features:**
- Runs a single test with maximum debug output
- Automatic failure type detection (crash/gas/behavior)
- Trace divergence analysis (PC, opcode, gas, stack)
- Next-step debugging guidance
- Quick reference commands

</details>

<details>
<summary><b>test-subset.ts - Test Subset Runner</b></summary>

```bash
# Using helper scripts
bun scripts/test-subset.ts Cancun
bun scripts/test-subset.ts transientStorage
bun scripts/test-subset.ts MCOPY

# Or using shell scripts
./scripts/test-subset.sh Cancun
./scripts/test-subset.sh "exact_test_name"

# Direct filtering with zig build
TEST_FILTER="Cancun" zig build specs
TEST_FILTER="transientStorage" zig build specs
TEST_FILTER="push0" zig build specs
```

**Use for:** Running entire test categories by hardfork, EIP, or pattern.

</details>

### Granular Spec Targets

Large hardforks are split into smaller sub-targets for faster iteration:

```bash
# Berlin
zig build specs-berlin-acl
zig build specs-berlin-intrinsic-gas-cost
zig build specs-berlin-intrinsic-type0
zig build specs-berlin-intrinsic-type1

# Frontier
zig build specs-frontier-precompiles
zig build specs-frontier-identity
zig build specs-frontier-create
zig build specs-frontier-call
zig build specs-frontier-calldata
zig build specs-frontier-dup
zig build specs-frontier-push
zig build specs-frontier-stack
zig build specs-frontier-opcodes

# Shanghai
zig build specs-shanghai-push0
zig build specs-shanghai-warmcoinbase
zig build specs-shanghai-initcode

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [evmts/xvi](https://github.com/evmts/xvi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
