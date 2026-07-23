---
trigger: always_on
description: This file provides development guidelines for AI agents working on the GoByte Core codebase.
---

# Agent Guidelines for GoByte Core

This file provides development guidelines for AI agents working on the GoByte Core codebase.

## Project Overview

GoByte is a cryptocurrency node implementation and direct fork of Dash (which itself is a fork of Bitcoin Core), written primarily in C++ with Python functional tests. The codebase uses autotools for building. It includes masternode features, LLMQ (Long Living Masternode Quorums), InstantSend, ChainLocks, and uses the **NeoScrypt** algorithm for PoW mining (unlike Dash's X11).

## Directory Structure

```
src/
├── bench/              # Performance benchmarks
├── bls/                # BLS cryptographic operations
├── coinjoin/          # CoinJoin mixing (replaces PrivateSend)
├── compat/             # Compatibility layer
├── consensus/         # Consensus rules (merkle, tx_verify)
├── crypto/            # Cryptographic primitives (sha256, neoscrypt)
├── evo/               # Evolution/deterministic masternode data
├── governance/        # Governance system (proposals, voting)
├── immer/             # Immutable data structures
├── interfaces/        # Wallet/node interfaces
├── leveldb/           # LevelDB database
├── llmq/              # LLMQ (Long Living Masternode Quorums)
├── masternode/        # Masternode infrastructure
├── policy/            # Policy (fees, policy.cpp)
├── primitives/        # Transaction/block primitives
├── qt/                # GUI implementation (Qt 5)
├── rpc/               # JSON-RPC server and endpoints
├── script/            # Script interpreter
├── secp256k1/         # Elliptic curve cryptography
├── support/           # Utility functions
├── test/              # Unit tests (Boost::Test)
├── univalue/          # JSON library
├── wallet/            # Wallet implementation
└── zmq/               # ZeroMQ notifications
```

### Directories to Exclude

**Under any circumstances**, do not make changes to:
- `guix-build*` - Build system files
- `releases` - Release artifacts
- Vendored dependencies:
  - `src/{leveldb,secp256k1,univalue,bls,immer}`
  - `src/crypto/ctaes` - AES encryption

**Unless specifically prompted**, avoid:
- `.github` - GitHub workflows and configs
- `depends` - Dependency build system
- `ci` - Continuous integration
- `contrib` - Contributed scripts
- `doc` - Documentation

## Build Commands

### Initial Setup
```bash
./autogen.sh
./configure [options]
make
```

### Common Configure Options
```bash
./configure --enable-debug          # Debug build with symbols
./configure --disable-tests          # Skip building tests
./configure --disable-bench          # Skip building benchmarks
./configure --enable-lcov            # Enable code coverage
./configure --with-sanitizers=address,undefined  # Address/undefined sanitizers
```

### Running Tests
```bash
make check                           # Run all unit tests
src/test/test_gobyte                 # Run tests directly
test_gobyte --run_test=suite_name    # Run specific test suite
test_gobyte --run_test=suite_name/test_case  # Run specific test case
test_gobyte --log_level=all --run_test=suite_name  # Verbose output
test_gobyte --help                  # Show available options
```

### Running Functional Tests
```bash
test/functional/test_runner.py          # Run all functional tests
test/functional/test_runner.py --extended   # Run extended tests
test/functional/test_runner.py feature_rbf.py  # Run specific test
test/functional/test_runner.py --coverage    # Track RPC coverage
test/functional/test_runner.py -j$(nproc)  # Parallel execution
```

### Linting
```bash
test/lint/all-lint.py            # Run all Python lint checks
test/lint/lint-whitespace.py     # Check for trailing whitespace and tabs
test/lint/lint-includes.py       # Check for duplicate includes, Boost deps, bracket syntax
test/lint/lint-include-guards.py # Verify include guards are present
test/lint/lint-format-strings.py # Check format string consistency
test/lint/lint-logs.py           # Ensure logs are terminated with \n
test/lint/lint-assertions.py     # Check for proper assertion usage in RPC
test/lint/lint-circular-dependencies.sh  # Detect circular dependencies
test/lint/lint-filenames.sh      # Verify filename conventions
test/lint/lint-python.py        # Python flake8 and mypy checks
test/lint/lint-shell.py         # Shell script linting with shellcheck
test/lint/lint-shell-locale.py  # Verify LC_ALL=C in shell scripts
test/lint/check-rpc-mappings.py # Validate RPC argument consistency
contrib/devtools/clang-format-diff.py   # Check C++ formatting
```

## C++ Code Style

### Formatting (clang-format)
- Use `src/.clang-format` for automatic formatting
- 4 space indentation (no tabs) for every block except namespaces
- No indentation for `public`/`protected`/`private` or `namespace`
- Braces on new lines for classes/functions/methods
- Braces on same line for control structures
- No extra spaces inside parentheses
- No space after function names; one space after `if`, `for`, `while`
- Single-line `if` without braces allowed; otherwise use braces

### Naming Conventions
- **Variables/functions**: snake_case (e.g., `my_variable`, `get_value`)
- **Class names**: UpperCamelCase (e.g., `MyClass`) - avoid C prefix

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gobytecoin/gobyte](https://github.com/gobytecoin/gobyte) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
