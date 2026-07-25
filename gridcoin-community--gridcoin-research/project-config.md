---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Gridcoin is a Bitcoin-derived proof-of-stake cryptocurrency that rewards users for contributing computational power to scientific research through BOINC. The codebase is C++17 with a Qt GUI, using CMake as the primary build system.

## Build Commands

### Development Build (Linux Native)

```bash
# Configure (from repo root)
cmake -B build -DENABLE_GUI=ON -DENABLE_TESTS=ON -DCMAKE_BUILD_TYPE=RelWithDebInfo

# Build
cmake --build build -j $(nproc)

# Executables: build/src/gridcoinresearchd, build/src/qt/gridcoinresearch
```

### Automated Build Script

```bash
./build_targets.sh TARGET=native BUILD_TYPE=RelWithDebInfo
# Other targets: depends (Linux static), win64 (Windows cross-compile)
```

### Tests

The `build_targets.sh` script runs the test suite automatically after building.

```bash
# Run all tests
ctest --test-dir build
# Or directly:
./build/src/test/test_gridcoin

# Run a specific test suite
./build/src/test/test_gridcoin --run_test=beacon_tests

# Run with verbose output
./build/src/test/test_gridcoin --log_level=all
```

### Linting

```bash
test/lint/lint-all.sh                    # All lint checks
test/lint/lint-whitespace.sh             # Trailing whitespace, tabs
test/lint/lint-include-guards.sh         # Header guard consistency
test/lint/lint-circular-dependencies.sh  # Circular #include detection
```

### Local CI (requires Docker + act)

```bash
./contrib/devtools/run-local-ci.sh workflow=.github/workflows/cmake_quality.yml job=lint
```

## Code Style

The project follows **modern Bitcoin Core coding standards** documented in `doc/developer-notes.md`. Key points:

- **Allman/ANSI brace style**, 4-space indentation, no tabs
- No space after function names; one space after `if`, `for`, `while`
- ClangFormat config at `src/.clang-format`
- **Older code** uses Hungarian notation (`nCount`, `strName`, `fEnabled`, `vItems`, `mapEntries`, `pPointer`). When working in an area with older style, use judgment about whether to match the local conventions or use the modern standard -- avoid gratuitous style churn in otherwise focused patches.

## Architecture

### Layer Model

```
User Interface:     Qt GUI (src/qt/)  |  RPC (src/rpc/)  |  CLI (gridcoinresearchd)
                                         |
Gridcoin Logic:     Contract System  |  Beacon  |  Tally/Accrual  |  Quorum/Superblock
                    (src/gridcoin/)
                                         |
Blockchain Core:    Block Validation  |  Staking  |  P2P Network  |  Wallet
                    (src/main.cpp, miner.cpp, net.cpp, wallet.cpp)
                                         |
Data/Network:       LevelDB  |  BDB (wallet)  |  Scraper (src/gridcoin/scraper/)
```

### Gridcoin-Specific Subsystems (src/gridcoin/)

These are what differentiate Gridcoin from Bitcoin:

- **Contract system** (`contract/`): Blockchain-stored governance actions (beacon, project whitelist, protocol params, votes). Contracts are special transactions dispatched to `IContractHandler` implementations via a registry pattern.
- **Beacon** (`beacon.h/cpp`): Links BOINC CPIDs to wallet keys for reward eligibility. Lifecycle: pending -> superblock activation -> active (6-month expiry).
- **Tally** (`tally.h/cpp`): Tracks per-CPID research reward accruals using periodic snapshots for O(1) lookups.
- **Superblock/Quorum** (`superblock.h/cpp`, `quorum.h/cpp`): Daily consensus snapshots of network research statistics. Scraper convergence (not voting) achieves consensus in current protocol.
- **Scraper** (`scraper/`): Distributed BOINC statistics collection. Active scrapers download project stats and publish signed manifests; subscriber nodes receive manifests and run convergence to build superblocks.
- **AutoGreylist** (`project.h/cpp`): Automatically excludes unresponsive BOINC projects based on Zero Credit Days (ZCD) and Whitelist Activity Score (WAS).
- **MRC** (`mrc.h/cpp`): Manual Research Claims for non-staking researchers.
- **Side Stakes** (`sidestake.h/cpp`): Automatic reward distribution to configured addresses.

### Key Bitcoin-Inherited Files (modified for Gridcoin)

| File | Role |
|------|------|
| `src/main.cpp` | Block/transaction validation, chain management (includes contract validation) |
| `src/miner.cpp` | Proof-of-stake block creation, research reward claiming, sidestake application |
| `src/net.cpp` | P2P networking |
| `src/wallet.cpp` | Wallet operations |
| `src/init.cpp` | Startup/shutdown orchestration |

### Registry Access Pattern

Gridcoin subsystems use singleton registries accessed via global functions:
```cpp
GetBeaconRegistry(), GetWhitelist(), GetProtocolRegistry(),
GetSideStakeRegistry(), GetScraperRegistry()
```

### Thread Architecture

Key threads: `ThreadStakeMiner` (block generation), `ThreadScraper`/`ThreadScraperSubscriber` (statistics collection, mutually exclusive), `ThreadSocketHandler`/`ThreadMessageHandler` (P2P), `ThreadRPCServer`. Full list in `doc/developer-notes.md`.

### Lock Ordering


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gridcoin-community/Gridcoin-Research](https://github.com/gridcoin-community/Gridcoin-Research) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
