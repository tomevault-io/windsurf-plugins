---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Solcore is an experimental implementation of Solidity's new type system. It's a prototype compiler that produces executable EVM code. The compiler implements a sophisticated type system with parametric polymorphism (generics) and type classes (similar to Haskell), which are compiled down to monomorphic code through specialization. This is later translated into lower level language (basically Yul with sums and products), from which Yul code is generated. So far we rely on external tools to translate Yul code to EVM bytecode.

**Important**: This is a research prototype, not production-ready. It contains bugs and is not optimized for UX.

## Build & Development Commands

### Setup
```bash
# Enter development shell with all dependencies (recommended)
nix develop

# If nix flakes give errors, add to ~/.config/nix/nix.conf:
# experimental-features = nix-command flakes
```

The development shell includes:
- Haskell tools: GHC 9.8, cabal, HLS
- Solidity tools: solc, foundry-bin, hevm
- C++ tools: cmake, boost (for testrunner)
- Utilities: jq, go-ethereum, goevmlab

### Build
```bash
# Build the project
cabal build

# Build with nix (runs full CI pipeline locally)
nix build

# Enter REPL for interactive development
cabal repl
```

### Testing
```bash
# Run all tests
cabal test

# Run specific test - the test suite uses tasty, individual tests can be filtered
cabal test --test-options="-p 'pattern'"

# Build C++ testrunner (required for contest integration tests)
cmake -S . -B build
cmake --build build --target testrunner
# Creates: build/test/testrunner/testrunner

# Run contest integration tests
export testrunner_exe=build/test/testrunner/testrunner
bash run_contests.sh

# Or run contest tests via Nix (builds everything and runs tests automatically)
nix flake check
```

### Compilation Pipeline

The compiler is split into **two separate binaries**:

1. **sol-core**: Typechecks, specializes, and lowers to Core IR
2. **yule**: Translates Core IR to Yul

```bash
# Compile .solc source to .core IR
cabal run sol-core -- -f <input.solc>
# Produces: output1.core

# Translate .core to .yul
cabal run yule -- output1.core -o output.yul

# Optional: skip deployment code generation
cabal run yule -- output1.core -o output.yul --nodeploy
```

### Running Contracts

Use `runsol.sh` for the full pipeline (sol-core → yule → solc → geth):

```bash
# Basic execution
./runsol.sh <file.solc>

# With function call
./runsol.sh <file.solc> --runtime-calldata "transfer(address,uint256)" "0x123..." "100"

# With raw calldata
./runsol.sh <file.solc> --runtime-raw-calldata "0xabcd..."

# Skip deployment (run runtime code directly)
./runsol.sh <file.solc> --create false

# Debug with interactive trace viewer
./runsol.sh <file.solc> --debug-runtime
./runsol.sh <file.solc> --debug-create

# Pass value (in wei)
./runsol.sh <file.solc> --runtime-callvalue 1000000000
```

## High-Level Architecture

### Compilation Pipeline Flow

```
Source (.solc) → Parser → AST → Early Desugaring → Type Checker → Late Desugaring → Core IR → Yul
                          ↑                                                            ↑
                                          Frontend (sol-core)                    Backend (yule)
```

The pipeline consists of 13 sequential passes (see `SolcorePipeline.hs:67-168`):

**Phase 1: Parsing & Early Desugaring (Untyped AST)**
1. **Parsing** → Parse source to untyped AST
2. **Name Resolution** → Resolve names and build AST (`CompUnit Name`)
3. **Field Access Desugaring** → Desugar contract field access syntax
4. **Contract Dispatch Generation** → Generate method dispatch code for contracts
5. **SCC Analysis** → Analyze strongly connected components for mutual recursion
6. **Indirect Call Handling** → Defunctionalization (eliminate higher-order functions)
7. **Wildcard Replacement** → Replace pattern wildcards with fresh variables
8. **Function Type Argument Elimination** → Remove function-typed parameters

**Phase 2: Type Checking**
9. **Type Inference** → Constraint-based bidirectional type checking → Typed AST (`CompUnit Id`)

**Phase 3: Late Desugaring & Lowering (Typed AST)**
10. **If/Bool Desugaring** → Lower if-expressions to pattern matching on sum types
11. **Match Compilation** → Compile complex patterns to simple case trees (Augustsson's algorithm)
12. **Specialization** → Monomorphization of polymorphic/overloaded code
13. **Core Emission** → Translate to Core IR (first-order functional IR)

**Phase 4: Yul Translation (Separate Binary)**
14. **Yul Translation** (`yule` binary) → Translate Core IR to Yul code (EVM-oriented assembly)

**Key Insight**: Early desugaring (steps 3-8) simplifies contract-specific syntax and higher-order constructs BEFORE type checking. This allows the type checker to work on a simpler, more uniform AST. Late desugaring (steps 10-11) handles constructs that benefit from type information.

### Key Modules

**Pipeline Orchestration**:
- `src/Solcore/Pipeline/SolcorePipeline.hs` - Main compilation pipeline (orchestrates all 13 passes)

**Phase 1: Parsing & Early Desugaring (Untyped)**:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [argotorg/solcore](https://github.com/argotorg/solcore) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->
