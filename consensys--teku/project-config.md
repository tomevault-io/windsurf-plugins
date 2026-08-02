---
trigger: always_on
description: This file provides guidance to AI assistants when working with code in this repository.
---

# AI Agent Instructions & Project Context

This file provides guidance to AI assistants when working with code in this repository.

## Project Overview

Teku is an open-source Ethereum consensus client written in Java, implementing a full beacon node and validator client. It is written in Java 25+ and follows the Ethereum consensus specifications.

## Build and Development Commands

### Building
```bash
# Full build with tests
./gradlew build

# Build without tests
./gradlew assemble

# Create distribution packages
./gradlew distTar installDist

# Build Docker image
./gradlew distDocker
```

### Code Style
```bash
# Apply Google Java code formatting (required before committing)
./gradlew spotlessApply

# Check code formatting
./gradlew spotlessCheck
```

### Testing
```bash
# Run all unit tests
./gradlew test

# Run tests for a specific module (example)
./gradlew :ethereum:spec:test

# Run integration tests
./gradlew integrationTest

# Run reference tests (consensus spec tests)
./gradlew referenceTest

# Run one reference suite manually (example)
ENV_TEST_TYPE=fork_choice/on_attestation ENV_SPEC=minimal ENV_MILESTONE=gloas ./gradlew --no-daemon :eth-reference-tests:referenceTest --tests tech.pegasys.teku.reference.ManualReferenceTestRunner -x generateReferenceTestClasses

# Run acceptance tests
./gradlew acceptanceTest

# Run property tests
./gradlew propertyTest
```

### Specrefs
Spec references live in `specrefs/` and reference the [Ethereum consensus specs](https://github.com/ethereum/consensus-specs/) or the user's local checkout of that repository. Run from the repository root:
```bash
ethspecify process --path=specrefs
ethspecify check --path=specrefs
```

When updating specrefs, keep exceptions only for references that are genuinely unimplemented, unnecessary, or intentionally implemented differently in Teku. If a reference has a real implementation source, remove it from exceptions and add stable `sources` anchors. After large generated diffs, check for duplicate entries:
```bash
rg '^- name:' specrefs/*.yml | sed 's/.*:- name: //' | sort | uniq -d
```

### Distribution
The built distribution is located in:
- Packaged: `build/distributions/`
- Expanded (ready to run): `build/install/teku/`

## Architecture Overview

### High-Level Architecture Diagram

```mermaid
graph TB
    subgraph "Application Layer"
        CLI[Teku CLI / BeaconNodeCommand]
        CLI --> Node[Node<br/>COMBINED/VC_ONLY/BOOTNODE]
    end

    subgraph "Service Layer"
        Node --> SC[ServiceController]
        SC --> SS[StorageService]
        SC --> ELS[ExecutionLayerService]
        SC --> BCS[BeaconChainService]
        SC --> PCS[PowchainService]
        SC --> VCS[ValidatorClientService]
        SC --> TS[TimerService]
    end

    subgraph "Core Logic Layer"
        BCS --> Spec[ethereum/spec<br/>Consensus Rules]
        BCS --> ST[ethereum/statetransition<br/>State Processing]
        BCS --> Sync[beacon/sync<br/>Block Sync]
        VCS --> ValClient[validator/client<br/>Duties & Slashing Protection]
        ELS --> ELM[ExecutionLayerManager<br/>Engine API / Builder API]
    end

    subgraph "Data & Storage Layer"
        SS --> DB[(Database<br/>RocksDB/LevelDB)]
        SS --> PA[storage/protoarray<br/>Fork Choice]
        BCS --> Storage[storage/api<br/>ChainStorage]
        Storage --> DB
    end

    subgraph "Networking Layer"
        BCS --> Net[networking/p2p<br/>Libp2p Gossip]
        Net --> Peers[Peer Network]
    end

    subgraph "API Layer"
        BCS --> RestAPI[data/beaconrestapi<br/>Beacon API]
        VCS --> ValAPI[validator/api<br/>Validator API]
    end

    subgraph "External Integration"
        ELM --> EL[Execution Layer Client<br/>Geth/Nethermind/Besu]
        ELM --> Builder[MEV-Boost Builder]
        PCS --> ETH1[Ethereum PoW Chain<br/>Deposit Contract]
    end

    subgraph "Event Channels (Event-Driven Communication)"
        TS -.SlotEventsChannel.-> BCS
        BCS -.VoteUpdateChannel.-> PA
        BCS -.CombinedStorageChannel.-> Storage
        ELS -.ExecutionClientEventsChannel.-> BCS
        PCS -.Eth1EventsChannel.-> BCS
    end

    subgraph "Infrastructure Layer"
        Infra[infrastructure/<br/>async, events, metrics, crypto, BLS, KZG, SSZ]
        Spec --> Infra
        ST --> Infra
        Storage --> Infra
        Net --> Infra
    end

    style CLI fill:#e1f5ff
    style Node fill:#e1f5ff
    style BCS fill:#ffe1e1
    style ELS fill:#ffe1e1
    style VCS fill:#ffe1e1
    style Spec fill:#fff4e1
    style DB fill:#e1ffe1
    style Net fill:#f0e1ff
    style RestAPI fill:#ffe1f5
    style EL fill:#f5f5f5
    style Infra fill:#e8e8e8
```

### Module Dependency Layers

```
┌─────────────────────────────────────────────────────────────┐
│  teku/                    (Application Entry Point)         │
└────────────────────────────┬────────────────────────────────┘
                             │
┌────────────────────────────┴────────────────────────────────┐
│  services/         (Service Orchestration Layer)            │
│  ├─ beaconchain, chainstorage, executionlayer               │
│  ├─ powchain, timer, bootnode, zkchain                      │
└────────────┬──────────────────────────┬─────────────────────┘

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Consensys/teku](https://github.com/Consensys/teku) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
