---
trigger: always_on
description: Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.
---

# Plutus Core Development Instructions

Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.

## Critical Development Environment Requirements

**MANDATORY: You MUST use Nix for development.** All builds and tests require the Nix environment. Do not attempt to build without Nix - it will fail due to missing dependencies and specific toolchain requirements.

### Nix Environment Setup

Install Nix first if not available:
```bash
curl -L https://nixos.org/nix/install | sh
```

Enter the development shell:
```bash
nix develop --no-warn-dirty --accept-flake-config
```

Your prompt will change to indicate you're in the Nix shell environment. This shell provides all required tools including:
- Correct GHC version with all Haskell dependencies
- Cabal build tool
- Haskell Language Server (HLS)
- Pre-commit hooks and formatting tools
- Documentation build tools

## Building the Project

### Essential Pre-Build Steps
```bash
# Always run these before building
cabal update
```

### Primary Build Commands

**CRITICAL BUILD TIMING - NEVER CANCEL BUILDS:**

Build the core library (15-30 minutes):
```bash
cabal build plutus-core  # NEVER CANCEL - takes 15-30 minutes, timeout: 45+ minutes
```

Build all packages (45-90 minutes):
```bash
cabal build all  # NEVER CANCEL - takes 45-90 minutes, timeout: 120+ minutes
```

Build specific packages:
```bash
cabal build plutus-tx           # 10-20 minutes
cabal build plutus-ledger-api   # 10-15 minutes  
cabal build plutus-tx-plugin    # 15-25 minutes
cabal build cardano-constitution # 5-10 minutes
cabal build plutus-executables  # 10-15 minutes
```

### Built Executables

The build produces these command-line tools:
```bash
cabal run plutus          # Plutus Core evaluator and tools
cabal run pir             # Plutus IR tools
cabal run plc             # Plutus Core compiler  
cabal run uplc            # Untyped Plutus Core tools
```

Example: Evaluate a Plutus Core program:
```bash
cabal run plc evaluate -- -h    # Show help for evaluation options
```

### Alternative GHC Versions

The project supports multiple GHC versions. Use specific shells:
```bash
nix develop --no-warn-dirty --accept-flake-config .#ghc96   # Primary (default)
nix develop --no-warn-dirty --accept-flake-config .#ghc98   # Alternative
nix develop --no-warn-dirty --accept-flake-config .#ghc910  # Alternative
```

## Testing

### Test Suite Execution

**CRITICAL TESTING TIMING - NEVER CANCEL TESTS:**

Run all tests (30-60 minutes):
```bash
cabal test all  # NEVER CANCEL - takes 30-60 minutes, timeout: 90+ minutes
```

Run specific test suites:
```bash
cabal test plutus-core-test           # 15-25 minutes
cabal test untyped-plutus-core-test   # 10-15 minutes
cabal test plutus-ir-test             # 10-15 minutes
cabal test plutus-tx-test             # 10-15 minutes
cabal test plutus-ledger-api-test     # 5-10 minutes
```

### Golden Test Regeneration

Regenerate all golden test files:
```bash
./scripts/regen-goldens.sh  # NEVER CANCEL - takes 45-75 minutes, timeout: 120+ minutes
```

For plugin tests with multiple GHC versions:
```bash
nix develop .#ghc96 --command cabal test plutus-tx-plugin --test-options=--accept
```

### Validation Scenarios

ALWAYS run these validation steps after making changes:

1. **Build validation**:
   ```bash
   cabal build all --ghc-options=-Werror  # Must pass with no warnings
   ```

2. **Test validation**:
   ```bash
   cabal test all  # All tests must pass
   ```

3. **Specific functionality tests**:
   ```bash
   # Test Plutus Core evaluator
   cabal run plc evaluate -- --help
   
   # Test conformance (if modifying core evaluator)
   cabal run haskell-conformance -- --help
   cabal run agda-conformance -- --help
   
   # Test benchmarks (if modifying performance-critical code)
   cabal run plutus-benchmark-nofib-tests -- --help
   ```

4. **Formatting validation**:
   ```bash
   stylish-haskell --config .stylish-haskell.yaml --check <modified-files>
   cabal-fmt --check <modified-cabal-files>
   ```

### Performance and Conformance Testing

The repository includes specialized test suites:

**Conformance tests** (validate evaluator correctness):
- `plutus-conformance/` - Compare Haskell vs Agda implementations
- Run with: `cabal test haskell-conformance` and `cabal test agda-conformance`

**Benchmark tests** (validate performance):
- `plutus-benchmark/` - Performance regression tests
- Run with: `cabal test plutus-benchmark-nofib-tests`

## Development Tools and Linting

### Pre-commit Hooks and Formatting

Pre-commit hooks are defined in the Nix shell configuration and run automatically on commit. The Nix environment provides the following formatting tools:

```bash
# These tools are available in the Nix shell:
stylish-haskell --config .stylish-haskell.yaml <files>  # Haskell formatting
cabal-fmt <cabal-files>                                  # Cabal file formatting
fourmolu --mode inplace <files>                          # Alternative Haskell formatter
```

If you commit outside the Nix shell, you may get:
```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [IntersectMBO/plutus](https://github.com/IntersectMBO/plutus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
