---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Forage** is a plugin extension for Apache Camel that provides opinionated bean factories for simplified component configuration. The library eliminates manual Java bean instantiation by providing factory classes configurable through properties files, environment variables, or system properties.

**Technology Stack:**
- Java 17+
- Apache Camel 4.x
- LangChain4j 1.x
- Maven, Spotless (Palantir Java Format), JUnit 5, AssertJ, Testcontainers, Citrus Test Framework

## Branching and Versioning Strategy

Forage uses **major.minor.micro** versioning (`1.4.0`, `1.4.1`, `1.6.0`, etc.), tracked
micro-to-micro against Apache Camel on the current-LTS line (`1.6.0` ↔ Camel `4.22.0`,
`1.6.1` ↔ Camel `4.22.1`, etc.).

| Branch | Tracks | Version | Status |
|--------|--------|---------|--------|
| `main` | Current Camel LTS | `1.6.x` | Follows the current Camel LTS release (currently 4.22.x). Active, primary branch — micro bumps (`1.6.0` → `1.6.1` → ...) for each release. |
| `camel-4.18.x` | Previous Camel LTS | `1.4.x` | Maintenance branch for the previous LTS line (Camel 4.18.x), branched from `main` when Camel 4.22 became LTS. Micro bumps (`1.4.1` → `1.4.2` → ...) as needed. |
| `camel-latest` | Latest (non-LTS) Camel | `1.5.x` | **Dormant.** Frozen at Camel 4.20.x / Forage 1.5.x until Apache Camel ships its next post-LTS "latest" release (expected around 4.23), at which point it's rebased from `main` and resumes releases. |

When a new Camel LTS lands, the current `main` line becomes the new `camel-N.M.x`
maintenance branch (via a fresh branch off `main`, not a rename), `main` is upgraded
in place to track the new LTS, and `camel-latest` is re-evaluated against whatever
Camel considers "latest" at that point.

### Backporting

When making changes (features, bug fixes, etc.), **always investigate whether the change needs backporting** to the other active branch(es):
- A fix on `main` may also apply to `camel-4.18.x` (and to `camel-latest` once it's active again), and vice versa.
- After completing work on one branch, check if the same change is relevant to the other branch(es) and create a backport PR if needed.
- Use `/oss-backport-pr` to automate backporting when applicable.

## Build Commands

```bash
# Full build with tests
mvn clean install

# Compile only (includes automatic code formatting)
mvn clean compile

# Apply code formatting manually
mvn spotless:apply

# Check code formatting
mvn spotless:check

# Run all tests
mvn verify

# Run a single test class
mvn test -Dtest=ClassName

# Run a single test method
mvn test -Dtest=ClassName#methodName

# Run integration tests for a specific module
mvn verify -f integration-tests/jdbc

# Run integration tests with specific runtime (plain, quarkus, spring-boot)
export INTEGRATION_TEST_RUNTIME=quarkus
mvn clean verify -f integration-tests/jdbc -Dit.test=JdbcTest

# Skip tests
mvn install -DskipTests
```

## Project Structure

```
forage/
├── core/                           # Core interfaces and utilities
│   ├── forage-core-ai/            # AI interfaces (ModelProvider, ChatMemoryFactory)
│   ├── forage-core-common/        # Config system (ConfigStore, ConfigModule, ConfigEntry, ConfigEntries, AbstractConfig)
│   ├── forage-core-vectordb/      # EmbeddingStoreProvider interface
│   ├── forage-core-jdbc/          # DataSourceProvider interface
│   ├── forage-core-jms/           # JMS interfaces
│   ├── forage-core-jta/           # JTA transaction interfaces
│   ├── forage-core-cloud/         # Cloud provider interfaces
│   └── forage-core-vertx/         # Vert.x interfaces
├── library/                        # Implementation modules
│   ├── ai/                        # AI implementations
│   │   ├── agents/                # forage-agent, forage-agent-factories
│   │   ├── chat-memory/           # Memory providers (message-window, infinispan, redis)
│   │   ├── models/chat/           # Model providers (openai, ollama, gemini, anthropic, etc.)
│   │   └── vector-dbs/            # Vector DB providers (qdrant, milvus, pgvector, etc.)
│   ├── jdbc/                      # JDBC data source providers
│   ├── jms/                       # JMS connection factories
│   ├── cloud/                     # Cloud provider implementations
│   └── vertx/                     # Vert.x implementations
├── integration-tests/              # Citrus-based integration tests
├── tests/plans/                    # End-to-end test plans (Markdown)
│   ├── common/                    # Shared procedures (container setup, forage-run)
│   ├── jdbc-datasource.md         # JDBC DataSource provisioning
│   ├── jms-messaging.md           # JMS ConnectionFactory provisioning
│   ├── cxf-soap-endpoints.md      # CXF/SOAP endpoint provisioning
│   ├── rabbitmq-connection.md     # Spring RabbitMQ provisioning
│   ├── property-validation.md     # Property typo detection and --strict mode
│   ├── config-commands.md         # camel forage config read/write
│   └── route-policies.md          # Flip and schedule route policies
├── tooling/                        # Build tooling

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [KaotoIO/forage](https://github.com/KaotoIO/forage) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-26 -->
