---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build System and Common Commands

Gradle with Kotlin DSL. Java 17+ required. All commands from project root.

```bash
./gradlew build                          # Build with tests
./gradlew build -xtest                   # Build without tests
./gradlew test                           # Run all tests
./gradlew test --tests "TestClassName"   # Run specific test class
./gradlew test --tests "io.prometheus.SomeTestClass.someTestMethod"  # Single test method
./gradlew --rerun-tasks check            # Force rerun all checks (lint + tests)
./gradlew agentJar proxyJar              # Generate standalone JARs
./gradlew generateProto                  # Regenerate protobuf stubs
./gradlew koverHtmlReport                # HTML coverage report (build/reports/kover/html/)
./gradlew koverXmlReport                 # XML coverage report (CI / Codacy / Coveralls)
./gradlew koverLog                       # Print coverage % to console
```

### Code Quality

```bash
./gradlew lintKotlinMain lintKotlinTest  # Run kotlinter linter
./gradlew detekt                         # Run detekt static analysis
./gradlew formatKotlin                   # Auto-format code
```

Always run lint and build before completing tasks:
`./gradlew detekt && ./gradlew lintKotlinMain && ./gradlew build -xtest`

### Useful Make Targets

```bash
make help            # List all targets with descriptions (auto-extracted from `## …` annotations)
make tests           # Rerun all checks (lint + tests)
make nh-tests        # Unit tests only (agent, proxy, common, misc — no harness)
make ip-tests        # In-process integration tests only
make netty-tests     # Netty integration tests only
make tls-tests       # TLS integration tests only
make container-tests # Full Testcontainers end-to-end suite (proxy + agent + nginx + Prometheus); needs Docker
make all-tests       # Full suite: `make tests` + `make container-tests`
make scaling-tests   # Parameter-driven scaling container test (tune via SCALE_* vars); needs Docker
make regen-certs     # Regenerate the testing/certs TLS fixtures (CA + server + client; 2048-bit)
make coverage        # Run tests + generate HTML and XML coverage reports
make coverage-xml    # XML coverage report (for Codacy/Coveralls/etc.)
make coverage-log    # Print coverage % to console
make tsconfig        # Regenerate ConfigVals from config/config.conf via tscfg
```

## Architecture

A **Prometheus Proxy** system enabling Prometheus to scrape metrics from endpoints behind firewalls.

### Request Flow

```
Prometheus → Proxy HTTP (:8080) → AgentContext lookup → ScrapeRequest via gRPC stream
    → Agent scrapes actual endpoint → ScrapeResponse via gRPC stream → Proxy → Prometheus
```

### Core Components

1. **Proxy (`io.prometheus.Proxy`)** — Runs outside the firewall alongside Prometheus
   - `ProxyGrpcService` — accepts agent connections on port 50051
   - `ProxyHttpService` / `ProxyHttpRoutes` — serves proxied metrics on port 8080
   - `ProxyPathManager` — maps URL paths to agent contexts
   - `AgentContextManager` — tracks connected agents
   - `ScrapeRequestManager` — manages scrape request lifecycle with timeouts
   - `ProxyServiceImpl` — implements the gRPC `ProxyService` definition

2. **Agent (`io.prometheus.Agent`)** — Runs inside the firewall with monitored services
   - `AgentGrpcService` — connects to proxy, streams scrape requests/responses
   - `AgentHttpService` — scrapes actual metrics endpoints using Ktor HTTP client
   - `AgentPathManager` — manages path registrations
   - `HttpClientCache` — caches HTTP clients keyed by auth credentials (TTL/idle eviction)

3. **Common (`io.prometheus.common/`)** — shared between proxy and agent
   - `BaseOptions` — CLI argument parsing and config loading (parent of `AgentOptions` / `ProxyOptions`)
   - `ConfigVals` — type-safe config wrapper (auto-generated from HOCON via tscfg; see `make tsconfig`)
   - `ScrapeResults` — scrape response data model
   - `EnvVars` — environment variable mappings

### Public API Surface (Dokka)

Only these types are part of the supported, documented public API. Everything else is `internal`:

- `io.prometheus.Agent` (entry point + companion `main` / `startSyncAgent` / `startAsyncAgent`)
- `io.prometheus.Proxy` (entry point + companion `main`)
- `io.prometheus.agent.AgentOptions` / `io.prometheus.proxy.ProxyOptions` / `io.prometheus.common.BaseOptions`
- `io.prometheus.agent.EmbeddedAgentInfo` (handle returned by `Agent.startAsyncAgent`)
- `io.prometheus.common.EnvVars`
- `io.prometheus.common.ConfigLoadException` (thrown by `startAsyncAgent` on a config-load failure when `exitOnMissingConfig` is false, so embedded hosts can catch it instead of the JVM exiting)

When promoting a type from `internal` to `public`, also add a cross-reference to it in `docs/packages.md` (the Dokka `includes.from` file). When demoting, remove the link to avoid dangling references in the generated site.

### gRPC Service Definition

Defined in `src/main/proto/proxy_service.proto`. Key RPCs:

- `readRequestsFromProxy` — server-streaming: proxy sends scrape requests to agent

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pambrose/prometheus-proxy](https://github.com/pambrose/prometheus-proxy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
