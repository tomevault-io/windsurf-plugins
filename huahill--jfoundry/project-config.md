---
trigger: always_on
description: This is a Java 25 multi-module Maven project for a jMolecules-based, runtime-neutral DDD framework. Top-level groups are declared in `pom.xml`: `jfoundry-core` contains domain, architecture, application, and infrastructure modules; `jfoundry-runtime` contains Spring, Quarkus, and Helidon; `jfoundry-dependencies` provides release dependency management. Production code uses standard Maven paths such as `src/main/java`; tests live under `src/test/java`; module resources live under `src/main/resourc
---

# Repository Guidelines

## Project Structure & Module Organization

This is a Java 25 multi-module Maven project for a jMolecules-based, runtime-neutral DDD framework. Top-level groups are declared in `pom.xml`: `jfoundry-core` contains domain, architecture, application, and infrastructure modules; `jfoundry-runtime` contains Spring, Quarkus, and Helidon; `jfoundry-dependencies` provides release dependency management. Production code uses standard Maven paths such as `src/main/java`; tests live under `src/test/java`; module resources live under `src/main/resources` or `src/test/resources`. SQL files shipped by jfoundry are copyable templates, not auto-run migrations. Documentation is organized by language under `docs/i18n/en/` and `docs/i18n/zh/`, with the default English overview in `README.md` and the Chinese overview in `README_ZH.md`.

## Build, Test, and Development Commands

- `mvn validate` checks the Maven reactor and module structure.
- `mvn test` compiles and runs all unit, integration, and ArchUnit tests.
- `mvn clean install` performs a full local build and installs artifacts into the local Maven repository.
- `mvn -pl jfoundry-domain test` runs tests for one module; add `-am` when dependencies must also be built.
- `scripts/verify-ci-matrix.sh` runs the local Java 25 release-baseline test using `JAVA_25_HOME`.
- `mvn clean install -DskipTests` builds artifacts without executing tests; use only for local iteration.

## Coding Style & Naming Conventions

Use Java 25 features where they simplify the model, especially records for immutable value objects. Follow the existing package root `org.jfoundry.*` and standard Maven layout. Keep domain modules free of Spring and persistence dependencies; place Spring auto-configuration in capability-specific modules under `jfoundry-runtime/jfoundry-spring/autoconfigure`, Spring runtime adapters under `jfoundry-runtime/jfoundry-spring/runtime`, persistence and broker adapters under `jfoundry-core/jfoundry-infrastructure`, reusable architecture test rules under `jfoundry-core/jfoundry-architecture/jfoundry-architecture-test`, and runtime-specific integration verification in the direct `jfoundry-<runtime>-integration-tests` module. Name tests with a `*Test` suffix. No formatter plugin is configured, so match the surrounding Java style: four-space indentation, clear method names, and concise English Javadocs/comments only where API intent or non-obvious behavior needs explanation.

Prefer stable Java 25 APIs that match the intended semantics. Use `ScopedValue` for lexically bounded dynamic
context; retain `ThreadLocal` only for genuinely mutable, thread-owned state and document non-obvious cases.
Catch `Error` only at boundaries that must clean up or invalidate state, rethrow it immediately, and preserve
the primary failure when cleanup also fails by attaching cleanup failures as suppressed exceptions. Shared
runtime-neutral or Jakarta adapters must not bind to a concrete runtime logging system. Runtime adapters should
use the logging facade guaranteed by their target runtime, so equivalent behavior need not use the same facade.

## Architecture Boundaries

JFoundry framework internals use Onion Simple: `domain`, `application`, and `infrastructure` are the dependency
rings. Runtime integrations are outer adapters; Hexagonal is an optional architecture language for downstream
projects, not the framework's internal module-placement model.

The core framework must remain independent of runtime frameworks such as Spring, Spring Boot, Quarkus, Helidon, Micronaut, CDI, and Jakarta EE runtime APIs. Keep these boundaries explicit:

- `jfoundry-domain` contains domain modeling primitives and must not depend on application, infrastructure, persistence, messaging, or runtime integration modules.
- `jfoundry-application` contains application-layer contracts, transaction abstractions, domain event orchestration, event externalization rules, Outbox/Inbox SPI, messaging SPI, and serialization SPI. It must not depend on Spring, MyBatis-Plus, broker clients, or concrete databases.
- `jfoundry-infrastructure` contains concrete adapters for persistence, messaging, serialization, and job execution. Infrastructure adapters may depend on native clients such as MyBatis-Plus, Kafka clients, RabbitMQ Java client, RocketMQ client, Jackson, or JobRunr, but they must not depend on Spring Framework or Spring Boot unless they are deliberately placed under `jfoundry-spring`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [huahill/jfoundry](https://github.com/huahill/jfoundry) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
