---
trigger: always_on
description: OLake project — architecture, conventions, build, and end-to-end testing guidance
---


# OLake — Cursor Rules

## 1. Project Overview

OLake is an open-source, high-performance **EL (Extract-Load)** tool that replicates data from databases, Kafka, and S3 into **Apache Iceberg** tables or **plain Parquet** files.

- **Languages:** Go 1.25.9 (primary) + Java 17 (Iceberg writer)
- **Go workspace:** `go.work` links the root module with 8 driver sub-modules
- **CLI:** Cobra-based commands: `spec`, `check`, `discover`, `sync`, `clear`
- **Sources:** Postgres, MongoDB, MySQL, Oracle, MSSQL, DB2, Kafka, S3
- **Destinations:** Apache Iceberg (via gRPC to Java), plain Parquet

## 2. Repository Layout

```
├── connector.go                     # RegisterDriver() entry point
├── go.work / go.mod                 # Go workspace + root module
├── build.sh                         # Build & run helper (auto-detects Iceberg JAR, DB2 env)
├── Makefile                         # gomod, golangci, gofmt, pre-commit
├── Dockerfile                       # Multi-stage build (Go + Java 17 runtime)
├── protocol/                        # CLI commands (spec, check, discover, sync, clear)
├── drivers/
│   ├── abstract/                    # Shared orchestration (backfill, CDC, incremental)
│   │   └── interface.go             # DriverInterface — every source implements this
│   └── <name>/                      # Each driver: main.go, go.mod, internal/, resources/spec.json
│       ├── docker-compose.yml       # Test infrastructure for that source
│       └── internal/                # Driver logic + *_test.go integration tests
├── destination/
│   ├── interface.go                 # destination.Writer — every destination implements this
│   ├── writers.go                   # WriterPool, WriterThread (concurrent writer management)
│   ├── iceberg/                     # Iceberg destination (legacy + arrow write modes)
│   │   ├── olake-iceberg-java-writer/   # Java Maven project (gRPC server)
│   │   ├── proto/                   # Protobuf definitions + generated Go code
│   │   ├── legacy-writer/           # Default: Go flattens → gRPC rows → Java writes
│   │   ├── arrow-writer/            # Arrow mode: Go → Parquet → Java registers
│   │   └── local-test/              # Docker Compose for local Iceberg (MinIO, Spark, JDBC catalog)
│   └── parquet/                     # Plain Parquet destination
├── types/                           # Core types (Stream, Catalog, State, Record, DataType)
├── utils/                           # Concurrency, flattening, type resolution, logging, telemetry
├── pkg/                             # Protocol packages (waljs, binlog, jdbc, kafka, parser)
└── constants/                       # Global constants, DriverType enum
```

## 3. Destinations — Iceberg & Parquet

### Iceberg Destination — Two Write Modes

- **Legacy (default):** Go flattens records → sends via gRPC `RecordIngestService` → Java writes to Iceberg
- **Arrow (`"arrow_writes": true`):** Go builds Arrow batches → Parquet files → gRPC `ArrowIngestService` → Java registers into catalog
- Java code: `destination/iceberg/olake-iceberg-java-writer/`

### Parquet Destination

- **Plain Parquet files:** Go writes directly to Parquet via the `destination/parquet` implementation — no Java writer or Iceberg catalog involved.
- **Output location:** Controlled entirely by the Parquet writer config (e.g., local or object storage path), with files organized by stream and batch for efficient downstream reads.
- **Config:** Set `"type":"PARQUET"` in the destination config to use this path; it uses the same `destination.Writer` interface as Iceberg but bypasses all Iceberg JAR and catalog setup.
   
  
### JAR Detection Order (IMPORTANT)

Both `build.sh` and the Go binary check for the JAR in this order:
1. **Project root:** `olake-iceberg-java-writer.jar` — if found, this is used and nothing else is checked
2. **Maven target:** `destination/iceberg/olake-iceberg-java-writer/target/olake-iceberg-java-writer-0.0.1-SNAPSHOT.jar`

**Stale JAR warning:** If `olake-iceberg-java-writer.jar` exists in the project root, `build.sh` will skip rebuilding entirely and the Go binary will use the stale copy. After rebuilding with Maven, you MUST either:
- Delete the root JAR first: `rm -f olake-iceberg-java-writer.jar`
- Or copy the fresh one: `cp destination/iceberg/olake-iceberg-java-writer/target/olake-iceberg-java-writer-0.0.1-SNAPSHOT.jar ./olake-iceberg-java-writer.jar`

### Rebuild procedure when Java code changes

```bash
rm -f olake-iceberg-java-writer.jar  # remove stale root JAR if present
cd destination/iceberg/olake-iceberg-java-writer && mvn clean package -DskipTests && cd -
# Optionally copy to root for build.sh:
cp destination/iceberg/olake-iceberg-java-writer/target/olake-iceberg-java-writer-0.0.1-SNAPSHOT.jar ./olake-iceberg-java-writer.jar
```

## 4. Build & Run

```bash
# Via build.sh (recommended — auto-detects Iceberg JAR, DB2 env)
bash build.sh driver-<name> sync --config config.json --destination writer.json --catalog streams.json

# Via go build directly
cd drivers/<name> && go build -o olake main.go
./olake sync --config ... --destination ... --catalog ... --state ...
```

**Makefile:** `make gomod` | `make golangci` | `make gofmt` | `make pre-commit`

## 5. Coding Conventions


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [datazip-inc/olake](https://github.com/datazip-inc/olake) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-04 -->
