---
trigger: always_on
description: This file provides the essential context an AI coding agent needs to work effectively in the L2-Relayer codebase.
---

# CLAUDE.md — Project Guide for AI Agents

This file provides the essential context an AI coding agent needs to work effectively in the L2-Relayer codebase.

## What Is This Project?

**Jovay Relayer** is the core middleware for the Jovay L2 Rollup system. It sits between Layer 1 (Ethereum) and Layer 2 (Jovay), driving the full Rollup lifecycle:

1. Polls L2 blocks from the Tracer Service
2. Aggregates blocks into Chunks and Batches
3. Submits Batches to L1 via EIP-4844 Blob transactions
4. Coordinates with Prover Controller to obtain TEE/ZK proofs
5. Commits proofs to L1 via EIP-1559 transactions
6. Relays cross-chain messages (L1 ↔ L2)

## Tech Stack

- **Language**: Java 21
- **Framework**: Spring Boot 3.5
- **Build**: Maven (multi-module)
- **Database**: MySQL 8.0+ (MyBatis-Plus ORM, Flyway migrations)
- **Cache/Lock**: Redis (distributed locking, nonce cache)
- **Blockchain**: Web3j (Ethereum JSON-RPC client)
- **gRPC**: Prover Controller communication
- **CLI**: Spring Shell (Admin CLI)

## Module Structure

```
L2-Relayer/
├── relayer-commons/     # Shared: models, enums, ABI wrappers, RollupSpecs, utilities
├── relayer-dal/         # Data access: MyBatis-Plus entities & mappers
├── relayer-app/         # Core application (main Relayer process)
├── query-service/       # REST API service for external queries
├── admin-cli/           # Spring Shell CLI for runtime operations
├── jovay-sign-service-spring-boot-starter/  # Tx signing (Web3j / KMS)
└── docker/              # Dockerfiles & compose.yaml
```

## Package Layout (relayer-app)

```
com.alipay.antchain.l2.relayer
├── config/          # Spring config classes (RollupConfig, ParentChainConfig, etc.)
├── engine/          # Distributed task scheduling
│   ├── core/        #   Dispatcher (leader election), Duty (task execution), ScheduleContext
│   ├── checker/     #   IDistributedTaskChecker implementations
│   └── executor/    #   Task executors (BlockPolling, BatchCommit, ProofCommit, etc.)
├── service/         # Business services
│   ├── IRollupService / RollupServiceImpl          # Rollup pipeline
│   ├── IReliableTxService / ReliableTxServiceImpl  # Transaction lifecycle
│   ├── IOracleService / OracleServiceImpl          # Gas oracle
│   └── IMailboxService / MailboxServiceImpl         # Cross-chain messaging
├── core/            # Domain logic
│   ├── layer2/      #   RollupAggregator, economic strategy, GrowingBatchChunksMemCache
│   ├── blockchain/  #   L1Client, L2Client, NonceManager, TxManager, GasPriceProvider
│   └── prover/      #   ProverControllerClient (gRPC)
├── dal/repository/  # Repository interfaces & implementations
├── metrics/         # OpenTelemetry metrics
└── utils/           # Utility classes (ConvertUtil, etc.)
```

## Key Classes to Know

| Class | Where | What It Does |
|-------|-------|-------------|
| `RollupAggregator` | `core/layer2/` | Block → Chunk → Batch pipeline, compression, DA version |
| `L1Client` | `core/blockchain/` | Builds & sends EIP-4844/1559 txs, queries Rollup contract |
| `ReliableTxServiceImpl` | `service/` | Tx lifecycle: confirmation, speed-up, resend, retry |
| `Dispatcher` | `engine/core/` | Leader election (Redis lock), round-robin task assignment |
| `Duty` | `engine/core/` | Polls task table, dispatches to executor thread pools |
| `ProverControllerClient` | `core/prover/` | gRPC client for proof request/retrieval |
| `RollupSpecs` | `relayer-commons` | Fork-based protocol versioning (Batch versions, DA versions) |
| `BatchHeader` | `relayer-commons` | 105-byte fixed header structure |
| `Batch` / `Chunk` / `BlockContext` | `relayer-commons` | Core data structures for Rollup |
| `BatchVersionEnum` | `relayer-commons` | BATCH_V0 / V1 / V2 definitions |
| `DaVersion` | `relayer-commons` | DA_0 / DA_1 / DA_2 encoding formats |
| `BlobsDaData` | `relayer-commons` | EIP-4844 Blob encoding logic |
| `RollupEconomicStrategy` | `core/layer2/economic/` | Green/Yellow/Red zone gas price strategy |

## Data Model Summary

- **Batch**: Top-level L1 submission unit. Contains BatchHeader (105 bytes) + Chunks + DA data.
- **Chunk**: Group of consecutive L2 blocks + their transactions.
- **BlockContext**: 40-byte per-block metadata (specVersion, blockNumber, timestamp, baseFee, gasLimit, numTx, numL1Msg).
- **ReliableTransaction**: Tracks every L1/L2 tx through states: TX_PENDING → TX_PACKAGED → TX_SUCCESS / TX_FAILED / BIZ_SUCCESS.

## Database

- ORM: **MyBatis-Plus** (entities in `relayer-dal/.../entities/`)
- Migrations: **Flyway** (SQL files in `relayer-app/src/main/resources/db/migration/`)
- Key tables: `batches`, `chunks`, `reliable_transaction`, `inter_bc_msg`, `rollup_number_record`, `active_node`, `distributed_task`, `oracle_request`, `batch_prove_request`, `l2_merkle_tree`

## Build & Run

```bash
# Build (requires JDK 21+, Maven 3.8+)
mvn clean package -DskipTests

# Run via Docker Compose (recommended)
docker build -f docker/Dockerfile-Relayer -t jovay-relayer .
docker build -f docker/Dockerfile-QS -t jovay-query-service .
cd docker && docker compose up -d

# Run standalone
cd relayer-app/target && tar -xzf l2-relayer.tar.gz && cd l2-relayer && ./bin/start.sh
```

## Configuration


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jovaynetwork/jovay-relayer](https://github.com/jovaynetwork/jovay-relayer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
