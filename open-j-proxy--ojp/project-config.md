---
trigger: always_on
description: This file provides guidance for GitHub Copilot working inside this repository. Read it before making any changes.
---

# Copilot Instructions for Open J Proxy (OJP)

This file provides guidance for GitHub Copilot working inside this repository. Read it before making any changes.

---

## Agent Behavior Guidelines

- Use simple language and simple examples to explain things.
- Be honest, even when the honest answer is "I don't know" or "this approach has problems."
- Look for the best technical solution, not just the most convenient one.
- Don't default to agreement — push back when something seems wrong or suboptimal.
- Proactively offer questions, opinions, suggestions, and concerns rather than waiting to be asked.
- When root-causing an issue or suggesting a solution, always report an honest confidence level — either as a percentage (0–100%) or as a label (Low / Medium / High) — and briefly explain what is driving that confidence or uncertainty.

---

## Java Runtime Requirement

**This project uses Java 21. Use the Java 21 runtime for all build and test tasks.**

| Context | Minimum Java |
|---|---|
| ojp-jdbc-driver (runtime) | Java 11 |
| ojp-server (runtime) | Java 21 |
| Development / CI build | Java 21 (required) |

The root `pom.xml` compiles with `source/target = 11` but the server module overrides this to 21. **Do not lower these targets.** Never suggest Java 8 or Java 17 as the build/test runtime; always use Java 21.

---

## What OJP Is

OJP is the **world's first open-source JDBC Type 3 driver**. It consists of two main deployable artefacts:

1. **ojp-server** – a standalone gRPC server that owns and controls the real database connection pools (HikariCP). Applications never connect directly to the database.
2. **ojp-jdbc-driver** – a JDBC 4.2-compliant driver that clients drop in. Instead of opening real connections, it makes gRPC calls to ojp-server.

```
[Java App] --JDBC--> [ojp-jdbc-driver] --gRPC/HTTP2--> [ojp-server] --JDBC--> [Database]
```

Supported databases: PostgreSQL, MySQL, MariaDB, Oracle, SQL Server, DB2, H2.

---

## Repository Layout

This is a **multi-module Maven project**. All modules share the parent `pom.xml` at the root.

| Module | Purpose |
|---|---|
| `ojp-grpc-commons` | Shared Protobuf/gRPC contracts (`.proto` files) |
| `ojp-jdbc-driver` | JDBC driver implementation |
| `ojp-server` | gRPC server, HikariCP pool management, session/transaction tracking |
| `ojp-datasource-api` | SPI interface: `ConnectionPoolProvider` |
| `ojp-datasource-hikari` | Built-in HikariCP implementation (priority 100) |
| `ojp-datasource-dbcp` | Built-in DBCP2 implementation (priority 10) |
| `ojp-xa-pool-commons` | XA-capable pool provider and `XAConnectionPoolProvider` SPI |
| `ojp-testcontainers` | OJP-specific Testcontainers support for integration tests |
| `spring-boot-starter-ojp` | Spring Boot auto-configuration / starter |

---

## Build Commands

Always use Java 21 when running these commands.

```bash
# Run lint only (fast, no compilation needed)
mvn checkstyle:check

# Build everything, skip tests
mvn clean install -DskipTests -Dgpg.skip=true

# Build a single module and its dependencies
mvn clean install -pl ojp-server -am -DskipTests -Dgpg.skip=true

# Verify compilation only (quick sanity check before committing)
mvn clean compile
```

---

## Pre-commit Requirements

- All code must compile successfully before committing.
- All Checkstyle (SonarLint) rules must pass — **never commit code that fails `mvn checkstyle:check`**.
- Run `mvn clean compile` to verify both lint and compilation — **never commit code that fails**.
- Ensure you are using Java 21 as the active runtime before building or testing.

---

## Testing

Most tests in `ojp-jdbc-driver` are **integration tests** that require a running OJP server. H2 tests are the fast, embedded option.

### Running tests locally

**Step 1 – Download JDBC drivers:**
```bash
cd ojp-server
bash download-drivers.sh
cd ..
```

**Step 2 – Start the OJP server (leave running):**
```bash
mvn verify -pl ojp-server -Prun-ojp-server
```

**Step 3 – Run tests:**
```bash
cd ojp-jdbc-driver
mvn test -DenableH2Tests=true
```

All database test flags are disabled by default:

| Flag | Database |
|---|---|
| `-DenableH2Tests=true` | H2 (embedded, fast) |
| `-DenablePostgresTests=true` | PostgreSQL |
| `-DenableMySQLTests=true` | MySQL |
| `-DenableMariaDBTests=true` | MariaDB |
| `-DenableCockroachDBTests=true` | CockroachDB |
| `-DenableOracleTests=true` | Oracle |
| `-DenableSqlServerTests=true` | SQL Server |

For IDE runs, always add `-Dfile.encoding=UTF-8 -Duser.timezone=UTC` as JVM arguments.

- Use JUnit 5. Follow the `shouldReturnXxxWhenYyy` naming convention.
- Prefer `ojp-testcontainers` for new tests over manually managed Docker databases.

---

## Code Style

- **Java conventions**: camelCase for variables/methods, PascalCase for classes.
- **Lombok**: Used throughout (`@Getter`, `@Setter`, `@Builder`, `@Slf4j`, etc.). Do not write getters/setters by hand.
- **Indentation**: 4 spaces.
- **Comments**: Only when necessary to explain non-obvious logic. Code should be self-documenting.
- **New dependencies**: Check license compatibility (Apache 2.0 or compatible required). Minimize additions.
- **No secrets or credentials** in committed code — use environment variables or Testcontainers.

---

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Open-J-Proxy/ojp](https://github.com/Open-J-Proxy/ojp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
