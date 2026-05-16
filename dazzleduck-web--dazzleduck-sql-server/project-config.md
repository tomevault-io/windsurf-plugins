---
trigger: always_on
description: This document provides comprehensive information about the DazzleDuck SQL Server project for AI assistants and developers.
---

# DazzleDuck SQL Server - Project Documentation

This document provides comprehensive information about the DazzleDuck SQL Server project for AI assistants and developers.

## Overview

DazzleDuck SQL Server is a high-performance remote DuckDB server that supports both Apache Arrow Flight SQL (gRPC) and RESTful HTTP protocols. It enables multiple users to connect remotely and execute queries through various client libraries.

**Key Features:**
- Dual Protocol Support: Arrow Flight SQL (port 59307) and RESTful HTTP API (port 8081)
- Arrow-Native data transfers for maximum performance
- JWT Authentication for secure access
- Remote DuckDB query execution with distributed execution support
- Delta Lake and Hive partition pruning

## Build & Development

### Requirements
- JDK 21 (server modules)
- JDK 11+ (client modules: client, client-grpc, common, logger, logback)
- Maven (wrapper included: `./mvnw`)

### Build Commands
```bash
# Compile all modules
./mvnw compile

# Clean build (skip tests)
./mvnw clean package install -DskipTests

# Run all tests
./mvnw test

# Run tests for specific module
./mvnw test -pl dazzleduck-sql-http

# Build Docker image (single arch — quick local dev)
# Apple Silicon (arm64):
./mvnw package -DskipTests jib:dockerBuild -pl dazzleduck-sql-runtime -Djib.architecture=arm64
# x86-64 (amd64) — requires amd64 base image:
./mvnw package -DskipTests jib:dockerBuild -pl dazzleduck-sql-runtime

# Build both arm64 + amd64 images and create a local multi-arch manifest
# (amd64 is skipped by default until dazzleduck/base-jre has an amd64 variant;
#  enable with -Dskip.docker.amd64=false once the base image supports it)
./mvnw verify -DskipTests -pl dazzleduck-sql-runtime --also-make
```

Images produced:
- `dazzleduck/dazzleduck:{version}-arm64` / `latest-arm64`
- `dazzleduck/dazzleduck:{version}-amd64` / `latest-amd64`  (when enabled)
- `dazzleduck/dazzleduck:{version}` / `latest` — local manifest list (both arches)

### Maven JVM Flags
Required for Arrow memory management:
```bash
export MAVEN_OPTS="--add-opens=java.base/sun.nio.ch=ALL-UNNAMED --add-opens=java.base/java.nio=ALL-UNNAMED --add-opens=java.base/sun.util.calendar=ALL-UNNAMED"
```

### Running Locally
```bash
./mvnw exec:java -pl dazzleduck-sql-runtime -Dexec.mainClass="io.dazzleduck.sql.runtime.Main" -Dexec.args="--conf warehouse=warehouse"
```

### Docker
```bash
docker run -ti -p 59307:59307 -p 8081:8081 dazzleduck/dazzleduck:latest --conf warehouse=/data
```

## Project Structure

```
dazzleduck-sql-server/
├── dazzleduck-sql-runtime/     # Main entry point, server startup orchestration
├── dazzleduck-sql-flight/      # Arrow Flight SQL server implementation
├── dazzleduck-sql-http/        # HTTP REST API (Helidon-based)
├── dazzleduck-sql-common/      # Shared utilities, type handling, config
├── dazzleduck-sql-commons/     # DuckDB utilities, connection pool, transformations
├── dazzleduck-sql-client/      # HTTP client implementation (JDK 11+)
├── dazzleduck-sql-client-grpc/ # gRPC/Flight SQL client implementation (JDK 11+)
├── dazzleduck-sql-login/       # JWT authentication service
├── dazzleduck-sql-search/      # Full-text search indexing
├── dazzleduck-sql-micrometer/  # Micrometer metrics forwarding
├── dazzleduck-sql-logger/      # SLF4J Arrow logging provider
├── dazzleduck-sql-logback/     # Logback appender for log forwarding
├── dazzleduck-sql-scrapper/    # Prometheus metrics scraping
├── example/                     # Sample data and configurations
├── ui/                          # Frontend UI (arrow-js-frontend)
└── warehouse/                   # Default data warehouse location
```

## Module Details

### dazzleduck-sql-runtime
**Entry point for the entire system**
- `Main.java` - CLI argument parsing, shutdown hooks
- `Runtime.java` - Server lifecycle, configuration loading
- Starts both HTTP and Flight SQL servers

### dazzleduck-sql-flight
**Arrow Flight SQL server**
- `DuckDBFlightSqlProducer.java` - Core producer implementing FlightSqlProducer
- `FlightSqlProducerFactory.java` - Factory for creating producers
- Statement caching with configurable TTL
- Supports bulk ingestion via queues

**Key Auth Classes:**
- `AdvanceJWTTokenAuthenticator.java` - JWT validation
- `AdvanceBasicCallHeaderAuthenticator.java` - Basic auth
- `ConfBasedCredentialValidator.java` - Config-based validation

**Auditing:**
- `MicroMeterFlightRecorder.java` - Metrics recording
- `StatementTrackingInfo.java` - Query execution tracking

### dazzleduck-sql-http
**RESTful HTTP API (Helidon)**

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/v1/login` | POST | Authenticate, get JWT token |
| `/v1/query` | GET/POST | Execute SQL queries — Arrow IPC (default) or TSV (via `Accept` header) |
| `/v1/plan` | POST | Generate query execution plan |
| `/v1/ingest` | POST | Ingest Arrow data to Parquet |
| `/v1/cancel` | POST | Cancel running query |
| `/v1/ui` | GET | Web UI dashboard |
| `/health` | GET | Health check |

**Query response formats**

`/v1/query` content-negotiates on the `Accept` header:

| `Accept` header | Response `Content-Type` | Notes |
|---|---|---|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dazzleduck-web/dazzleduck-sql-server](https://github.com/dazzleduck-web/dazzleduck-sql-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
