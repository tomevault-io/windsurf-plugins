---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

CBOMkit generates, stores, views and compliance-checks **CBOMs** (Cryptography Bills of Materials, CycloneDX). Two deployables live in one repo:

- **Backend** (`src/`) — Quarkus 3 / Java 21 REST + WebSocket API, Postgres via Hibernate Panache.
- **Frontend** (`frontend/`) — Vue 2 + Carbon Design SPA. Also ships standalone as *CBOMkit-coeus* (viewer-only mode, `VUE_APP_VIEWER_ONLY=true`).

The actual source-code scanning is **not** in this repo: it comes from the `org.pqca:cbomkit-lib` dependency (indexing + scanner services for Java/Python/Go, aka CBOMkit-hyperion / sonar-cryptography). This repo orchestrates clone → index → scan → persist → serve.

## Build prerequisite

`org.pqca:cbomkit-lib` is hosted on **GitHub Packages**, so Maven needs a `~/.m2/settings.xml` with a `github` server entry (username + PAT with `read:packages`). Without it every build fails at dependency resolution — that is the first thing to check on a resolution error, not the pom.

## Commands

```shell
# backend
./mvnw quarkus:dev                       # dev mode, port 8081 (needs Postgres, see below)
./mvnw clean package                     # build (runs spotless+checkstyle+tests)
./mvnw test                              # tests only
./mvnw test -Dtest=GitServiceTest        # single test class
./mvnw test -Dtest=GitServiceTest#method # single test method
./mvnw spotless:apply                    # format + insert license headers
./mvnw spotless:check
./mvnw checkstyle:check

# frontend (cd frontend/)
npm ci
npm run serve      # dev server on :8001
npm run build
npm run lint

# environments (docker-compose profiles; ENGINE=podman also works)
make dev           # postgres only          -> run backend + frontend locally
make dev-backend   # postgres + frontend    -> run backend locally
make dev-frontend  # postgres + backend     -> run frontend locally
make production    # full stack
make ext-compliance# full stack + OPA container serving opa/*.rego
make coeus         # frontend only (viewer)
```

**Tests need a running Postgres.** All meaningful tests are `@QuarkusTest` and boot the app against `jdbc:postgresql://localhost:5432/postgres` (user/pass `cbomkit`). Start `make dev` first, or `mvn package` will fail on DB connect. Some tests (`GitServiceTest`, `PurlResolverTest`, `MavenPackageFinderService`) also hit the network.

`make` targets resolve `VERSION` by curling the **latest GitHub release tag**, not the pom version — so `make production` runs released images, and `make build-backend-image` tags the locally built jar with the latest release tag.

## Formatting is enforced at build time

The spotless plugin binds `apply` (not `check`) to the `validate` phase, and checkstyle binds `check` there too. Any `./mvnw` invocation will silently reformat your Java sources (google-java-format, **AOSP style**, 4-space indent) and inject the Apache/PQCA license header. New `.java` files without the header are fixed automatically; don't hand-write it.

## Architecture

Layered DDD + CQRS, built on the `app.bootstrap.core` library (`AggregateRoot`, `Repository`, `ICommandBus`, `ProcessManager`, `Projector`, `DomainEventHandler`).

```
presentation/  JAX-RS resources + WebSocket endpoint  (com.ibm.presentation.api.v1)
usecases/      commands, queries, handlers, process managers, projectors, services
domain/        aggregates, value objects, domain events — no framework deps
infrastructure/ buses, Panache entities, read models, compliance services, config
```

**ArchUnit enforces domain isolation** (`src/test/java/com/ibm/architecture/DomainTest.java`). `com.ibm.domain` may only depend on an explicit allowlist (`java..`, `app.bootstrap.core.ddd..`, `jakarta.annotation/inject`, `org.cyclonedx.model..`, `com.github.packageurl..`, `org.pqca.scanning..`). Adding any other import to a domain class breaks the build — either move the code to `usecases`/`infrastructure` or deliberately extend the allowlist.

### Scan flow (the core of the system)

1. `ScanningResource` (POST `/api/v1/scan`) or `WebsocketScanningResource` (`/v1/scan/{clientId}`) creates a `ScanId`, instantiates a **per-scan** `ScanProcessManager`, and registers it on the `CommandBus` for the scan's command types. The WebSocket path passes a `WebSocketProgressDispatcher` so the UI gets live progress; the REST path uses `EmptyProgressDispatcher`.
2. `RequestScanCommandHandler` builds the `ScanAggregate`, which emits `ScanRequestedEvent` (git URL) or `PurlScanRequestedEvent` (PURL).
3. `ScanEventHandler` turns those domain events into the first command (`CloneGitRepositoryCommand` / `ResolvePurlCommand`).
4. `ScanProcessManager` is the saga: resolve PURL → clone (JGit) → identify package folder → index modules → scan, sending the next command to itself via the bus after each step and persisting aggregate state in between. Every handler starts with `if (this.scanId != command.id()) return;` because *all* registered process managers see every command. On failure it dispatches an `ERROR` progress message and calls `compensate`.
5. `ScanFinishedEvent` → `CBOMProjector` writes the read model.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cbomkit/cbomkit](https://github.com/cbomkit/cbomkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
