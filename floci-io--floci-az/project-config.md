---
trigger: always_on
description: Guidance for AI coding agents working in the floci-az repository.
---

Guidance for AI coding agents working in the floci-az repository.

This file defines repository-specific operating rules for autonomous or semi-autonomous coding agents. Follow these instructions unless a maintainer explicitly tells you otherwise.

---

## Project Overview

floci-az is a Java-based local Azure emulator built on Quarkus.

Its goal is full Azure SDK and Azure CLI compatibility through real Azure wire protocols, not convenience APIs or simplified abstractions.

floci-az is the Azure counterpart of Floci (the AWS emulator) and floci-gcp. They share design patterns and Docker infrastructure, but are independent repositories.

- Port: **4577** (all HTTP services share this single port)
- AMQP port: **5672** (Event Hubs, managed by Artemis sidecar)
- Kafka port: **9093** (Event Hubs Kafka, managed by Redpanda sidecar, opt-in)
- Stack:
  - Java 25
  - Quarkus 3.x
  - JUnit 5 + RestAssured
  - Jackson
  - docker-java for sidecar container management

---

## First Principles

When making changes, follow these priorities:

1. Preserve Azure protocol compatibility
2. Match Azure SDK and CLI behavior
3. Reuse existing floci-az patterns
4. Prefer correctness over convenience
5. Keep changes narrow and testable

Critical rules:

- Do not introduce custom endpoint shapes
- Do not change request or response formats for convenience
- Do not perform broad refactors unless the task explicitly requires them
- Keep behavior aligned with Azure SDK expectations and existing floci-az conventions

---

## Architecture

floci-az follows a layered design:

- **AzureRoutingFilter** — pre-matching JAX-RS filter; extracts account name and service type from the request path; dispatches to the correct `AzureServiceHandler`
- **AzureServiceHandler** — interface implemented by each service (`getServiceType()`, `canHandle()`, `handle()`)
- **AzureServiceRegistry** — CDI registry of all handlers; checks `isEnabled()` per service
- **Service Handler** — parses Azure protocol input, contains business logic, produces Azure-compatible responses
- **StorageBackend** — pluggable persistence (memory / persistent / hybrid / wal)

### Core Infrastructure

- `EmulatorConfig` — SmallRye `@ConfigMapping`; prefix `floci-az`
- `AzureRoutingFilter` — path-based routing (suffix detection: `-queue`, `-table`, `-functions`, `-appconfig`, `-keyvault`, `-eventhub`)
- `AzureServiceRegistry` — handler discovery + `isEnabled()` per service type
- `BannerLogger` — startup banner listing all enabled services
- `StorageBackend` + `StorageFactory` — pluggable storage
- `XmlBuilder` — fluent XML builder with attribute support (`startAttr`, `selfClose`)
- `XmlParser` — StAX-based XML parser (no extra dependencies)
- `XmlUtils` — Jackson-based XML serialisation for structured models

### Sidecar Container Infrastructure (`core/docker/`)

Used by services that delegate to a managed Docker container:

- `ContainerSpec` — immutable container descriptor (record)
- `ContainerBuilder` — fluent `ContainerSpec` builder (network, ports, mounts, log rotation, DNS)
- `ContainerLifecycleManager` — create/start/stop/remove containers; volume management; endpoint resolution
- `ImageCacheService` — pull-once-per-image with registry credential support
- `PortAllocator` — thread-safe free TCP port allocation
- `ContainerDetector` — detects whether floci-az is running inside Docker
- `CurrentContainerNetworkResolver` — resolves which Docker network floci-az is on
- `DockerHostResolver` — resolves the correct host for child containers to reach floci-az
- `DockerClientProducer` — CDI producer for `DockerClient`

---

## Package Layout

```
io.floci.az.config           — EmulatorConfig
io.floci.az.core             — routing, registry, banner, XML utils, auth, DNS
io.floci.az.core.auth        — auth pipeline, verifiers
io.floci.az.core.docker      — container infrastructure
io.floci.az.core.storage     — storage backends
io.floci.az.services.<svc>   — per-service handlers
```

Typical service structure:

```
services/<svc>/
  *Handler.java       — implements AzureServiceHandler
  *Models.java        — domain objects (optional)
```

Sidecar-based services additionally have:

```
services/<svc>/
  *ContainerManager.java   — @ApplicationScoped, @PostConstruct/@PreDestroy lifecycle
  *Manager.java            — manages one sidecar (pull, start, health-wait, stop)
  *ConfigGenerator.java    — generates sidecar config file from EmulatorConfig
```

Rule: copy an existing service pattern before introducing a new one.

---

## Azure Protocol Rules

| Service | Protocol | Path Suffix | Notes |
|---|---|---|---|
| Blob Storage | Azure Storage REST | `/{account}/` | XML responses, Shared Key auth |
| Queue Storage | Azure Storage REST | `/{account}-queue/` | XML responses |
| Table Storage | Azure Storage REST | `/{account}-table/` | JSON / OData |
| Azure Functions | HTTP REST | `/{account}-functions/` | ZIP deploy, HTTP trigger invoke |
| App Configuration | Azure App Config REST | `/{account}-appconfig/` | JSON, ETags, labels |
| Key Vault | Azure Key Vault REST | `/{account}-keyvault/` | JSON, Bearer auth challenge |
| Event Hubs | AMQP 1.0 (Artemis) | port 5672 | Direct sidecar, not proxied through 4577 |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [floci-io/floci-az](https://github.com/floci-io/floci-az) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->
