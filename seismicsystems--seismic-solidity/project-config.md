---
trigger: always_on
description: Fork of the [Ethereum Solidity compiler](https://github.com/ethereum/solidity) that adds **confidential storage** to the EVM, enabling smart contracts to handle sensitive data privately on-chain. Upstream is tracked through the `develop` branch.
---

# Seismic Solidity (ssolc)

Fork of the [Ethereum Solidity compiler](https://github.com/ethereum/solidity) that adds **confidential storage** to the EVM, enabling smart contracts to handle sensitive data privately on-chain. Upstream is tracked through the `develop` branch.

## What This Does

Standard EVM storage is publicly readable by anyone. Seismic extends the compiler with four **shielded types** — `suint`, `sint`, `saddress`, `sbool` — that behave like their normal counterparts but store data in confidential storage slots. The compiler automatically emits `CSTORE`/`CLOAD` opcodes (instead of `SSTORE`/`SLOAD`) for shielded variables, so developers write familiar Solidity while getting privacy guarantees at the storage layer. Shielded types each occupy a full storage slot (no packing), cannot be `public`/`constant`/`immutable`, and cannot appear in events. Shielded arrays and mappings with shielded values are supported; shielded types as mapping keys or array indices are disallowed.

## Contributing

- **Pull requests** must target [`SeismicSystems/seismic-solidity`](https://github.com/SeismicSystems/seismic-solidity) against the `seismic` branch (the default branch) unless otherwise specified.
- Do not open PRs against `develop` (that tracks upstream Ethereum Solidity) or any other branch without explicit instruction.
- Do not open pull requests that only fix typos — they will be closed.

## Build

C++ project using CMake. The output binary is `build/solc/solc` (branded as `ssolc`).

### macOS (arm64/x86_64)

```bash
# Dependencies
brew install cmake boost

# Build
mkdir -p build && cd build
cmake ..
make -j$(sysctl -n hw.ncpu)
```

### Linux (Ubuntu)

```bash
# Dependencies
sudo apt-get update
sudo apt-get install -y build-essential cmake python3 zlib1g-dev libboost-all-dev libssl-dev

# Build
mkdir -p build && cd build
cmake ..
make -j$(nproc)
```

### Verify

```bash
build/solc/solc --version
# Expected: ssolc, the seismic solidity compiler commandline interface
# Version: 0.8.31-develop...
```

## Test

### Unit tests (soltest)

Runs Boost C++ unit tests (excludes semantic tests by default).

```bash
./scripts/soltest.sh
```

Filter to specific tests:

```bash
./scripts/soltest.sh -t 'syntaxTests/viewPureChecker/*'
```

### Semantic tests (requires seismic-revm)

Semantic tests run via `revme` from [seismic-revm](https://github.com/SeismicSystems/seismic-revm). Semantic test commands must be run from within the seismic-revm repo. All Seismic repos live as siblings under a shared workspace directory (see the workspace CLAUDE.md one level up for the full layout). Replace `<solidity-repo-root>` with the absolute path to your seismic-solidity checkout (e.g. for git worktrees, use the worktree path) and `<seismic-revm-repo-root>` with the absolute path to your seismic-revm checkout.

All configurations use `--unsafe-via-ir` to bypass a compile-time restriction — this does not force all tests through the via-IR pipeline.

**Without optimizer, without --via-ir:**

```bash
cd <seismic-revm-repo-root> && cargo run -p revme -- semantics \
  --keep-going --unsafe-via-ir \
  -s "<solidity-repo-root>/build/solc/solc" \
  -t "<solidity-repo-root>/test/libsolidity/semanticTests"
```

**With optimizer, without --via-ir:**

```bash
cd <seismic-revm-repo-root> && cargo run -p revme -- semantics \
  --keep-going --unsafe-via-ir \
  --optimize --optimizer-runs 200 \
  -s "<solidity-repo-root>/build/solc/solc" \
  -t "<solidity-repo-root>/test/libsolidity/semanticTests"
```

**Without optimizer, with --via-ir:**

```bash
cd <seismic-revm-repo-root> && cargo run -p revme -- semantics \
  --keep-going --unsafe-via-ir --via-ir \
  -s "<solidity-repo-root>/build/solc/solc" \
  -t "<solidity-repo-root>/test/libsolidity/semanticTests"
```

**With optimizer, with --via-ir:**

```bash
cd <seismic-revm-repo-root> && cargo run -p revme -- semantics \
  --keep-going --unsafe-via-ir --via-ir \
  --optimize --optimizer-runs 200 \
  -s "<solidity-repo-root>/build/solc/solc" \
  -t "<solidity-repo-root>/test/libsolidity/semanticTests"
```

Some tests may only fail with the optimizer enabled or disabled. Test both configurations when debugging issues.

### Interactive test expectation tool (isoltest)

`isoltest` manages syntax/analysis test expectations. Build it from the build directory:

```bash
cd build && make -j$(nproc) isoltest
```

**Always** pass `--no-semantic-tests` — semantic tests are run via `revme`, not isoltest.

```bash
# Run specific test(s)
build/test/tools/isoltest --no-semantic-tests -t "syntaxTests/types/shielded_*"

# Run all syntax tests
build/test/tools/isoltest --no-semantic-tests -t "syntaxTests/*"
```

`--accept-updates` can batch-fix test expectations, but **never run it without explicit approval** — it silently rewrites every failing test's expected output, which can mask regressions. Always review changes via `git diff` afterward.

### Quick compile check

```bash
echo 'pragma solidity ^0.8.0; contract T { suint256 private x; }' | build/solc/solc --bin -
```

## Project Layout

```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SeismicSystems/seismic-solidity](https://github.com/SeismicSystems/seismic-solidity) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-05 -->
