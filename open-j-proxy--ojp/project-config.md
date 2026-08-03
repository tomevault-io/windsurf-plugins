---
trigger: always_on
description: Manages regular (non-XA) connection pools. Built-in implementations: HikariCP (default, priority 100), DBCP2 (priority 10). To replace the pool, place a JAR implementing this interface in `ojp-libs/` — no recompile needed.
---

# Agents.md — AI Agent Guide for Open J Proxy (OJP)

This file provides guidance for AI coding agents (GitHub Copilot, etc.) working inside this repository. Read it before making any changes.

---

## Agent Behavior Guidelines

- Use simple language and simple examples to explain things.
- Be honest, even when the honest answer is "I don't know" or "this approach has problems."
- Look for the best technical solution, not just the most convenient one.
- Don't default to agreement — push back when something seems wrong or suboptimal.
- Proactively offer questions, opinions, suggestions, and concerns rather than waiting to be asked.
- When root-causing an issue or suggesting a solution, always report an honest confidence level — either as a percentage (0–100%) or as a label (Low / Medium / High) — and briefly explain what is driving that confidence or uncertainty.

---

## What OJP Is

OJP is the **world's first open-source JDBC Type 3 driver**. It consists of two main deployable artefacts:

1. **ojp-server** – a standalone gRPC server that owns and controls the real database connection pools (HikariCP). Applications never connect directly to the database.
2. **ojp-jdbc-driver** – a JDBC 4.2-compliant driver that clients drop in. Instead of opening real connections, it makes gRPC calls to ojp-server.

The value proposition is back-pressure / connection-storm prevention: many application instances can scale elastically without ever overwhelming the database, because the proxy enforces a global connection limit.

```
[Java App] --JDBC--> [ojp-jdbc-driver] --gRPC/HTTP2--> [ojp-server] --JDBC--> [Database]
```

Supported databases (tested): PostgreSQL, MySQL, MariaDB, Oracle, SQL Server, DB2, H2.  
In principle any database that ships a JDBC driver should work.

---

## Repository Layout

This is a **multi-module Maven project**. All modules share the parent `pom.xml` at the root.

| Module | Purpose |
|---|---|
| `ojp-grpc-commons` | Shared Protobuf/gRPC contracts (`.proto` files). Both driver and server depend on this. |
| `ojp-jdbc-driver` | JDBC driver implementation (`OjpDriver`, `OjpConnection`, `OjpStatement`, `OjpResultSet`, …) |
| `ojp-server` | gRPC server, HikariCP pool management, session/transaction tracking, slow-query segregation |
| `ojp-datasource-api` | SPI interface: `ConnectionPoolProvider` |
| `ojp-datasource-hikari` | Built-in HikariCP implementation of `ConnectionPoolProvider` (priority 100) |
| `ojp-datasource-dbcp` | Built-in DBCP2 implementation of `ConnectionPoolProvider` (priority 10) |
| `ojp-xa-pool-commons` | XA-capable pool provider (`CommonsPool2XAProvider`) and `XAConnectionPoolProvider` SPI |
| `ojp-testcontainers` | OJP-specific Testcontainers support for reproducible integration tests |
| `spring-boot-starter-ojp` | Spring Boot auto-configuration / starter |

Documentation lives under `documents/`. ADRs are in `documents/ADRs/`. The `ROADMAP.md` at the root describes the release plan (1.0.0 targets September/October 2026).

---

## Java Version Requirements

| Context | Minimum Java |
|---|---|
| ojp-jdbc-driver (runtime) | Java 11 |
| ojp-server (runtime) | Java 25 |
| Development / CI build | Java 25 (recommended) |

The root `pom.xml` compiles with `source/target = 11` but the server module overrides this to 25. **Do not lower these targets.** CI tests against Java 11, 17, 21, and 25 for the driver.

---

## Build Commands

```bash
# Build everything, skip tests (required before running any tests)
mvn clean install -DskipTests -Dgpg.skip=true

# Build a single module and its dependencies
mvn clean install -pl ojp-server -am -DskipTests -Dgpg.skip=true

# Verify compilation only (quick sanity check before committing)
mvn clean compile
```

**Never commit code that fails `mvn clean compile`.**

---

## Testing

### Architecture of tests

Most tests in `ojp-jdbc-driver` are **integration tests**: they require a running OJP server and, for non-H2 databases, a running database container. This design choice means the tests verify the real end-to-end flow, which is appropriate for a proxy that sits between a driver and a database.

### Running tests locally

**Step 1 – Download JDBC drivers** (needed since 0.4.0-beta; drivers are no longer bundled):

```bash
cd ojp-server
bash download-drivers.sh        # downloads H2, PostgreSQL, MySQL, MariaDB to ojp-server/ojp-libs/
cd ..
```

**Step 2 – Start the OJP server** (leave this terminal open):

```bash
mvn verify -pl ojp-server -Prun-ojp-server
```

**Step 3 – Run tests** (in another terminal):

```bash
cd ojp-jdbc-driver
mvn test -DenableH2Tests=true   # H2 is embedded; no external DB needed
```

All database test flags are disabled by default. Enable only what you need:

| Flag | Database |
|---|---|
| `-DenableH2Tests=true` | H2 (embedded, fast) |
| `-DenablePostgresTests=true` | PostgreSQL |
| `-DenableMySQLTests=true` | MySQL |
| `-DenableMariaDBTests=true` | MariaDB |
| `-DenableCockroachDBTests=true` | CockroachDB |
| `-DenableOracleTests=true` | Oracle |
| `-DenableSqlServerTests=true` | SQL Server |


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Open-J-Proxy/ojp](https://github.com/Open-J-Proxy/ojp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
