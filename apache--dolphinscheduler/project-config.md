---
trigger: always_on
description: Apache DolphinScheduler is a distributed, visual DAG workflow-scheduling platform. This is the monorepo: backend servers (master / worker / api / alert), a Vue 3 frontend, plugin families for tasks / datasources / storage / alerting / scheduling, and the release tooling.
---

# AGENT.md - Apache DolphinScheduler

Apache DolphinScheduler is a distributed, visual DAG workflow-scheduling platform. This is the monorepo: backend servers (master / worker / api / alert), a Vue 3 frontend, plugin families for tasks / datasources / storage / alerting / scheduling, and the release tooling.

**This file is an agent-facing project index, adapted from `CLAUDE.md`.** Module-specific details currently live in each module's `CLAUDE.md`; use those files as the source of truth and do not duplicate module contents here.

---

## Tech stack (project-wide)

- **Java 1.8** (do not assume 11+ APIs; `dolphinscheduler-api-test` is the only Java 11 island).
- **Spring Boot 2.6.1** across servers, **Jetty** (Tomcat is excluded transitively).
- **MyBatis-Plus** for ORM; **HikariCP** for the metadata DB pool, **Druid** inside user-facing datasource plugins.
- **Quartz** for cron scheduling (via `scheduler-plugin`).
- **Netty / gRPC** for inter-server RPC (see `extract-base`).
- **Vue 3 + Vite + TypeScript + Naive UI** for the frontend.
- **Maven** multi-module reactor (26 modules in root `pom.xml` + 2 test modules).
- **Zookeeper 3.8** by default for the registry (Etcd and JDBC also supported).

## Runnable services

A production deployment runs **four independent services** (plus an external registry and metadata DB). A fifth entry point, `StandaloneServer`, embeds all four in one JVM for development.

| Service | Module | Main class | Default ports |
|---------|--------|------------|---------------|
| **API** | [`dolphinscheduler-api`](dolphinscheduler-api/CLAUDE.md) | `org.apache.dolphinscheduler.api.ApiApplicationServer` | `12345` (HTTP / UI + REST) |
| **Master** | [`dolphinscheduler-master`](dolphinscheduler-master/CLAUDE.md) | `org.apache.dolphinscheduler.server.master.MasterServer` | `5679` (RPC) |
| **Worker** | [`dolphinscheduler-worker`](dolphinscheduler-worker/CLAUDE.md) | `org.apache.dolphinscheduler.server.worker.WorkerServer` | `1235` (RPC) |
| **Alert** | [`dolphinscheduler-alert`](dolphinscheduler-alert/CLAUDE.md) (to `-alert-server`) | `org.apache.dolphinscheduler.alert.AlertServer` | `50053` (HTTP), `50052` (RPC) |
| Standalone (dev only) | [`dolphinscheduler-standalone-server`](dolphinscheduler-standalone-server/CLAUDE.md) | `org.apache.dolphinscheduler.StandaloneServer` | `12345` + `50052` (API + alert; master/worker use in-JVM calls) |

Every service is a `@SpringBootApplication` on Jetty and implements `IStoppable`. Scale Master / Worker / Alert horizontally; coordination happens via the registry (Zookeeper by default). API is stateless and also scales horizontally behind a load balancer.

Ports are overridable via `server.port` / service-specific keys in each service's `application.yaml`.

## Build & run

```bash
# Full build (release profile; produces dist tarball)
./mvnw clean install -Prelease

# Zookeeper 3.4 legacy
./mvnw clean install -Prelease -Dzk-3.4

# Skip UI build (faster iteration on backend only)
./mvnw -pl '!dolphinscheduler-ui' clean install

# Build one module (+ its required siblings)
./mvnw -pl dolphinscheduler-master -am clean install

# Format (Spotless is configured)
./mvnw spotless:apply

# Standalone server (after building)
cd dolphinscheduler-standalone-server/target && ./bin/start.sh
```

Binary artifact: `dolphinscheduler-dist/target/apache-dolphinscheduler-*-bin.tar.gz`.

## Test

```bash
# Unit tests for one module
./mvnw -pl dolphinscheduler-master test

# API integration tests (separate reactor, requires Docker)
mvn -pl dolphinscheduler-api-test/dolphinscheduler-api-test-case test

# E2E browser tests (Selenium + Docker)
mvn -pl dolphinscheduler-e2e/dolphinscheduler-e2e-case test

# Apple Silicon: add -Dm1_chip=true to the Docker-driven suites
```

---

## Module index

Click into a module's `CLAUDE.md` for details. Each description is one line here on purpose.

### Core execution

- [`dolphinscheduler-master`](dolphinscheduler-master/CLAUDE.md) - workflow orchestration engine; consumes `Command`s, runs the DAG state machine, dispatches to workers.
- [`dolphinscheduler-worker`](dolphinscheduler-worker/CLAUDE.md) - runs physical tasks dispatched from master; hosts task plugins.
- [`dolphinscheduler-task-executor`](dolphinscheduler-task-executor/CLAUDE.md) - reusable task-lifecycle framework embedded by the worker.
- [`dolphinscheduler-alert`](dolphinscheduler-alert/CLAUDE.md) - alert server + channel plugins (email, Feishu, DingTalk, ...).

### API layer

- [`dolphinscheduler-api`](dolphinscheduler-api/CLAUDE.md) - REST API server (entry point for UI, Python SDK, external clients).
- [`dolphinscheduler-api-test`](dolphinscheduler-api-test/CLAUDE.md) - integration tests against the REST API (Docker Compose + Testcontainers).
- [`dolphinscheduler-authentication`](dolphinscheduler-authentication/CLAUDE.md) - Actuator-endpoint auth + AWS credential helpers (NOT the main login path).

### Shared libraries

- [`dolphinscheduler-common`](dolphinscheduler-common/CLAUDE.md) - foundation utilities (everything depends on this).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [apache/dolphinscheduler](https://github.com/apache/dolphinscheduler) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
