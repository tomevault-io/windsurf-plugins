---
trigger: always_on
description: 1. Direct user instructions (highest)
---

# MockServer — Agent Instructions

## Instruction Priority

1. Direct user instructions (highest)
2. Rules in `.opencode/rules/`
3. This file (`AGENTS.md`)
4. Skills in `.opencode/skills/`

## Project Overview

MockServer is an open-source HTTP(S) mock server and proxy for testing, written in Java. It uses Netty as the HTTP server framework, Maven for builds, and is deployed as Docker containers, JARs, and WARs.

**Tech stack:** Java 17+ (minimum supported), Netty 4.2, Jackson 2.22, Maven (multi-module), Node.js/TypeScript (UI + client), Python 3.9+ (client), Ruby 3.0+ (client), Docker, Helm, Jekyll (documentation site)
**CI/CD:** Buildkite (primary CI), GitHub Actions (Docker image builds, CodeQL)
**Infrastructure:** AWS (Buildkite build agents, documentation site hosting), Docker Hub (container images)
**Repository:** GitHub (github.com)

### Local Development Environment

**Docker is available locally.** Docker Desktop runs on the developer Mac, so Docker is available to agents in this environment (not only in CI). This means:

- **Docker-gated tests CAN and SHOULD be run locally**, not just in CI. Tests that guard on `Assume.assumeTrue(DockerAvailability.isAvailable(...))` (Testcontainers live-broker tests, `NET_ADMIN` transparent-proxy e2e, QUIC/HTTP-3 client tests, etc.) will actually execute here — validate them by running and passing them, not merely by confirming they skip.
- **Keep the Docker-gating in place anyway.** The `assumeTrue(...)` guard is still the correct design so the suite degrades gracefully on any CI agent or machine without Docker. Docker being present locally changes how we *validate*, not how we *write* the tests.
- **`org.mockserver.test.DockerAvailability` (in `mockserver-testing`) is the canonical availability probe. NEVER call `DockerClientFactory.instance().isDockerAvailable()` directly.** Despite its javadoc ("true if Docker is available, false if not") that method converts only `IllegalStateException` into `false` — it **throws** for everything else, because it also starts the Ryuk reaper and runs version/mount checks. A `BadRequestException` (privileged Ryuk rejected by a user-namespace-remapped daemon), a `ContainerFetchException`, or an `Error` such as `NoClassDefFoundError` from an incomplete test classpath all escape it, turning "no usable Docker" into a hard ERROR and defeating the `assume` guard — and Testcontainers caches that failure and rethrows it for the rest of the JVM. Write:
  ```java
  Assume.assumeTrue("Docker is not available",
      DockerAvailability.isAvailable(() -> DockerClientFactory.instance().isDockerAvailable()));
  ```
  Pass a **lambda, not a method reference** — `instance()` must be evaluated inside the wrapper's try/catch. Testcontainers is the preferred harness, and works with Testcontainers 1.21.4+ (docker-java 3.4.2) on Docker Desktop 4.67 / Engine 29.x / API 1.54. (Earlier versions — Testcontainers 1.20.6 / docker-java 3.4.1 — got a 400 on the info endpoint and the probe returned false even though Docker worked.)
- **A Docker-gated suite that runs in CI needs a fail-closed assertion too.** A fail-safe probe means an unusable Docker SKIPS rather than errors, which is right off-CI but is a silent false positive in CI. Pair it with `.buildkite/scripts/steps/assert-suite-ran.sh` over the suite's surefire reports so a skip fails the build loudly.
- `docker` CLI commands (`docker build`, `docker run`) are also available for Dockerfile smoke checks in the commit workflow.

**Behind a corporate TLS-inspection proxy?** If local dependency downloads fail with TLS / `certificate verify failed` errors, every toolchain needs to trust the corporate root CA (via a combined bundle), configured ONLY in your user/shell environment — never in repo or pipeline files. Set `LOCAL_DOCKER_CA_BUNDLE` for anything run through `.buildkite/scripts/run-in-docker.sh`, and per-toolchain env / `~/.npmrc` for host builds. Full reusable setup (new-laptop checklist + per-toolchain table): [docs/operations/build-system.md → Local Development Behind a Corporate TLS-Inspection Proxy](docs/operations/build-system.md#local-development-behind-a-corporate-tls-inspection-proxy).

### Project Documentation

Comprehensive internal documentation is maintained in `docs/`. **Always consult these docs before making changes** to understand architecture, conventions, and dependencies:

| Document | When to consult |
|----------|----------------|
| [docs/README.md](docs/README.md) | Documentation index and quick reference |
| [docs/code/overview.md](docs/code/overview.md) | Before modifying any module — understand module boundaries and dependencies |
| [docs/code/netty-pipeline.md](docs/code/netty-pipeline.md) | Before modifying Netty handlers, protocol detection, or TLS |
| [docs/code/request-processing.md](docs/code/request-processing.md) | Before modifying mock matching, proxy forwarding, or action dispatch |
| [docs/code/event-system.md](docs/code/event-system.md) | Before modifying event logging, verification, or persistence |
| [docs/code/memory-management.md](docs/code/memory-management.md) | Before modifying maxLogEntries, maxExpectations, ring buffer sizing, or memory defaults |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mock-server/mockserver-monorepo](https://github.com/mock-server/mockserver-monorepo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
