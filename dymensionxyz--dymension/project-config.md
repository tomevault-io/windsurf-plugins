---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is the Dymension Hub repository (`d-dymension`), which serves as the settlement layer of the Dymension protocol. It's a Cosmos SDK-based blockchain written in Go.

## FACTS

- Cosmos SDK code all runs in a single thread, there is no multithreading
- Transactions rollback if there is any error, in this sense they are atomic: all or nothing. If they panic, nothing happens.
- BeginBlocker and Endblocker functions in module should never panic, it would cause a chain halt
- All cosmos validators must get the same result so the code should be deterministic

## Common Development Commands

### Building and Installation
```bash
# Install dymd binary to $GOPATH/bin
make install

# Build dymd binary to ./build/dymd
make build

# Build debug version (with debug symbols, no optimization)
make build-debug

# Generate protobuf files (requires Docker)
make proto-gen

# Generate swagger documentation
make proto-swagger-gen

# Format proto files
make proto-format

# Lint proto files
make proto-lint
```

### Local Development Setup
```bash
# Setup a local node with default configuration
./scripts/setup_local.sh

# Start local node
dymd start

# Bootstrap liquidity pools
sh scripts/pools/pools_bootstrap.sh

# Setup incentives streams
sh scripts/incentives/fund_incentives.sh
```

### Testing
```bash
# Run all tests
go test ./...

# Run tests with race detection and coverage
go test -race -coverprofile=coverage.txt ./...

# Run specific module tests
go test ./x/rollapp/...

# Run integration tests (in ibctesting directory)
go test ./ibctesting/...
```

### Linting and Code Formatting
```bash
# Run golangci-lint (configured in .golangci.yml)
golangci-lint run

# Linters enabled: errcheck, gocyclo, gosec, govet, ineffassign, misspell, revive, staticcheck, unconvert, unused, errorlint
# Uses golangci-lint v2.1 (as per CI)

# Format Go code with gofumpt
gofumpt -w .

# Format proto files
make proto-format
```

## Architecture Overview

### Core Modules (`x/`)

#### Core Modules
- **rollapp**: RollApp registration, state updates, finalization queue, fraud proofs
  - Handles state info submissions from sequencers
  - Supports genesis bridges for RollApp initialization
  
- **sequencer**: Sequencer lifecycle (bonding/unbonding), rotation, rewards
  - Manages proposer selection and slashing
  
- **lightclient**: Canonical IBC light clients for RollApps (critical for EIBC)
  - Validates light client parameters against expected values
  
- **delayedack**: Custom IBC middleware for delayed packet acknowledgments
  - Manages packet finalization based on RollApp state updates
  - Handles fraud detection and rollback
  
- **eibc**: Fast finality via market makers fulfilling orders before finalization
  - Manages demand orders and liquidity providers
  
- **bridgingfee**: Fee management for Hub<->RollApp bridging
- **forward**: Token routing between Hyperlane and IBC/EIBC
- **kas**: Kaspa<->Dymension Hyperlane bridge bookkeeping

#### Economic Modules
- **iro**: Initial RollApp Offering with bonding curves
  - Handles vesting and liquidity provisioning
  
- **incentives**: Gauge-based liquidity rewards
  - Supports asset-specific and RollApp-specific gauges
  
- **sponsorship**: Validator-voted reward distribution
- **streamer**: Scheduled token releases
- **lockup**: Token locking for incentive rewards

#### Utility Modules
- **denommetadata**: IBC and Hyperlane token metadata
- **dymns**: .dym domain name service and marketplace
  - Supports aliases and reverse resolution
  
- **gamm**: AMM pools (from Osmosis)

### Key Architectural Patterns

- **Module Structure**: Standard Cosmos SDK with `keeper/`, `types/`, `client/cli/`
- **IBC Integration**: Custom middleware for RollApp packet handling
- **Hook System**: Cross-module communication via hooks
- **Testing**: Unit tests, keeper test suites, integration tests in `ibctesting/`

## Module Dependencies

Key inter-module dependencies:
- `rollapp` → `sequencer`: For sequencer validation
- `delayedack` → `rollapp`: For state finalization checks
- `eibc` → `delayedack`: For packet fulfillment
- `incentives` → `lockup`, `sponsorship`: For reward distribution
- `lightclient` → `rollapp`, `sequencer`: For canonical client validation

## GitHub Actions and CI/CD

### Workflows
The repository uses GitHub Actions for continuous integration. Key workflows include:

1. **Build and Test** (`test.yml`): Runs on all pushes and PRs
   - Builds the project
   - Runs tests with race detection and coverage
   - Uploads coverage to Codecov

2. **Linting** (`golangci_lint.yml`): Runs golangci-lint on all code
   - Uses golangci-lint v2.1
   - Configuration in `.golangci.yml`

3. **Protocol Buffers** (`proto.yaml`): Validates protobuf files
   - Runs proto-format and proto-gen
   - Checks for uncommitted changes
   - Breaking change detection with buf

4. **E2E Tests** (`e2e_test.yml`, `e2e_test_upgrade.yml`): Integration tests
   - Tests various scenarios including upgrades
   - Nightly runs for comprehensive testing

### Verifying CI Checks Locally

```bash
# Tests with coverage (mimics CI)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dymensionxyz) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
