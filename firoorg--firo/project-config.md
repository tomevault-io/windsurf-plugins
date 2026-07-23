---
trigger: always_on
description: Firo is a privacy-focused cryptocurrency forked from Bitcoin Core, featuring zero-knowledge proof protocols (Spark, Lelantus), masternode infrastructure (LLMQ), and FiroPOW mining. Current version: **0.14.15.3**. Licensed under MIT.
---

# CLAUDE.md - Firo Development Guide

## Project Overview

Firo is a privacy-focused cryptocurrency forked from Bitcoin Core, featuring zero-knowledge proof protocols (Spark, Lelantus), masternode infrastructure (LLMQ), and FiroPOW mining. Current version: **0.14.15.3**. Licensed under MIT.

## Build System

**CMake 3.22+** with **C++20** standard. No autotools.

### Quick Build (Linux)

```bash
# 1. Build dependencies
make -C depends -j$(nproc)

# 2. Configure and build
export HOST_TRIPLET=$(depends/config.guess)
cmake -G Ninja \
  -DCMAKE_TOOLCHAIN_FILE=$(pwd)/depends/$HOST_TRIPLET/toolchain.cmake \
  -DBUILD_TESTS=ON -DBUILD_GUI=ON -DENABLE_CRASH_HOOKS=ON \
  -DCMAKE_BUILD_TYPE=Release \
  -S. -Bbuild
cd build && ninja
```

### Key CMake Options

| Option | Default | Description |
|--------|---------|-------------|
| `BUILD_DAEMON` | ON | Build `firod` |
| `BUILD_GUI` | ON | Build `firo-qt` (requires Qt 6.7.3+) |
| `BUILD_CLI` | ON | Build `firo-cli` |
| `BUILD_TX` | `${BUILD_CLI}` | Build `firo-tx` (defaults to same as `BUILD_CLI`) |
| `BUILD_TESTS` | OFF | Build unit test suite |
| `ENABLE_WALLET` | ON | Wallet functionality |
| `WITH_ZMQ` | ON | ZeroMQ notifications |
| `ENABLE_CRASH_HOOKS` | OFF | Stack trace generation (auto-enabled for Release/RelWithDebInfo/MinSizeRel) |
| `CLIENT_VERSION_IS_RELEASE` | false | Release build flag |

### Build Outputs

Binaries go to `build/bin/`: `firod`, `firo-cli`, `firo-qt`, `firo-tx`

### Cross-Compilation

Use the `depends/` system with host triplets:
- Linux: `x86_64-pc-linux-gnu` (default), `aarch64-linux-gnu`
- Windows: `make -C depends HOST=x86_64-w64-mingw32`
- macOS: requires SDK in `depends/SDKs/`

### Docker Build

```bash
docker build . -t firo-local
docker run -d --name firod -v "${HOME}/.firo:/home/firod/.firo" firo-local
```

## Testing

### Unit Tests (Boost.Test)

```bash
cmake -Bbuild -DBUILD_TESTS=ON ...
cd build && ctest --output-on-failure
```

Test source: `src/test/` (~80 test files). Framework setup in `src/test/test_bitcoin.h`.

### Integration Tests (Python)

```bash
# Copy binaries where the test harness expects them
cp -rf build/bin/* build/src/
qa/pull-tester/rpc-tests.py -extended
```

Test scripts: `qa/rpc-tests/` (100+ Python scripts covering wallet, privacy protocols, masternodes, consensus).

### Test Networks

- `-testnet` for multi-node testing over the network
- `-regtest` for local single-node testing with on-demand block creation

## Repository Structure

```
src/                        # Main source code
├── libspark/               # Spark protocol (current privacy protocol) - ZK proofs, crypto primitives
├── spark/                  # Spark wallet integration and state management
├── liblelantus/            # Lelantus protocol (legacy privacy protocol)
├── wallet/                 # Full wallet implementation
├── rpc/                    # JSON-RPC API endpoints
├── qt/                     # Qt GUI wallet
├── evo/                    # Deterministic masternode lists, special transactions
├── llmq/                   # Long Living Masternode Quorums (chainlocks, instant send)
├── bls/                    # BLS signatures for quorum signing
├── bip47/                  # BIP47 payment codes
├── hdmint/                 # Hierarchical deterministic minting
├── crypto/                 # Cryptographic functions (SHA, HMAC, ChaCha20, Lyra2Z, ProgPoW)
├── consensus/              # Consensus rules and validation parameters
├── primitives/             # Block and transaction primitives
├── script/                 # Bitcoin script interpreter
├── policy/                 # Transaction policy (fees)
├── secp256k1/              # ECDSA library (subtree)
├── leveldb/                # Key-value storage (subtree)
├── univalue/               # JSON parsing (subtree)
├── test/                   # Unit tests
├── fuzz/                   # Fuzz testing
├── bench/                  # Benchmarks
├── config/                 # Build configuration headers
├── validation.cpp/h        # Core block/transaction validation (~6100 lines)
├── net.cpp/h               # Network layer
├── net_processing.cpp/h    # Peer protocol handling
├── init.cpp/h              # Application initialization
├── miner.cpp/h             # Block mining (FiroPOW)
├── pow.cpp/h               # Proof-of-Work consensus
├── chainparams.cpp/h       # Network parameters (mainnet/testnet/regtest)
└── firo_params.h           # Protocol parameters
depends/                    # Deterministic dependency build system
cmake/                      # CMake modules
contrib/                    # Auxiliary tools (gitian, guix, packaging, devtools)
qa/                         # Integration test suite
doc/                        # Documentation (build guides, developer notes, API docs)
share/                      # UI resources, scripts
```

## Coding Conventions

### Style Rules (from `src/.clang-format` and `doc/developer-notes.md`)

- **Indentation**: 4 spaces, no tabs
- **Braces**: Linux style - new line for namespaces, classes, functions; same line for control flow
- **No column limit** (ColumnLimit: 0)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [firoorg/firo](https://github.com/firoorg/firo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
