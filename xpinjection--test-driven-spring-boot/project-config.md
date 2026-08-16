---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Test Commands

```bash
# Build (runs all tests)
./mvnw clean package

# Run all tests (requires Docker for Testcontainers)
./mvnw test

# Run a single test class
./mvnw -Dtest=BookServiceTest test

# Run without Docker — skips every Testcontainers-based test
./mvnw verify -Dtestcontainers.enabled=false

# Run with Pact Broker contract verification (off by default)
./mvnw verify -Dpactbroker.enabled=true -Dpact.provider.version=0.1.0 -Dpact.provider.branch=master -Dpact.verifier.publishResults=true
```

**Add `--offline` to any Maven command when dependencies haven't changed** — it is a noticeable speedup.

Test execution notes:
- There is **no Failsafe / separate integration-test phase**. Surefire runs everything matching `**/*Test.java` and `**/*Rules.java` during the `test` phase, with `forkCount=0` (single JVM, shared Spring context cache). `verify` therefore runs the same tests as `test`, just with packaging and the CycloneDX SBOM on top.
- Docker-dependent tests are gated by `@DisabledIfSystemProperty(named = "testcontainers.enabled", matches = "false")`. The property must be explicitly `false` to skip them — absence means enabled.
- The Pact provider test (`BookApiPactTest`) is inversely gated on `pactbroker.enabled=true` and is skipped otherwise.

## Tech Stack

Java 25, Spring Boot 3.5.7, Lombok, PostgreSQL 17 + Flyway, Freemarker (UI), Spring Security, SpringDoc OpenAPI. Observability: Micrometer + Prometheus registry, Micrometer Tracing over OpenTelemetry with the **OTLP** exporter (not Zipkin), Zalando Logbook + logstash-logback-encoder for HTTP/JSON logging. Tests: JUnit 5, AssertJ, Mockito, RestAssured, Testcontainers, Database Rider, ArchUnit, Pact, Spring REST Docs, HtmlUnit, openapi-diff, swagger-coverage.

## Architecture

Hexagonal (Ports & Adapters), **mechanically enforced** by ArchUnit in `HexagonalDesignRules.java` — violations fail the build, so check the rules before restructuring anything.

```
domain/                    → core business entities (currently only Book)
service/                   → service INTERFACES only (must be interfaces named *Service)
  impl/                    → @Service implementations (*ServiceImpl)
  dto/                     → service-layer DTOs
  exception/               → business exceptions
adaptors/
  api/                     → inbound REST: *RestController, CommonErrorHandler, api-only DTOs
  persistence/             → outbound: *Dao (CrudRepository), entity/ subpackage
  ui/                      → outbound: *Controller rendering Freemarker templates
config/                    → Spring config, LibrarySettings, init/ startup listeners
```

**Layer access matrix** (who may access whom — everything else is a violation):

| Layer | May be accessed by |
|---|---|
| `domain` | `service.impl`, `adaptors.persistence` |
| `service` (+ `dto`, `exception`) | `service.impl`, `adaptors.api`, `adaptors.ui`, `config` |
| `adaptors.persistence` | `service.impl` only |
| `service.impl`, `adaptors.api`, `adaptors.ui`, `config` | nobody |

**Naming rules** (enforced): `@Entity` → `..domain..` or `..adaptors.persistence.entity..`; `CrudRepository` → `..adaptors.persistence` + `*Dao`; `@RestController` → `..adaptors.api` + `*RestController`; `@Controller` → `..adaptors.ui` + `*Controller`; `@Service` → `..service.impl` + `*ServiceImpl`; everything directly in `..service` must be an interface named `*Service`.

Note the deliberate asymmetry: `Book` is a domain entity, while `ExpertEntity` is a persistence-adaptor entity. Both are legal — pick the one that matches whether the type is genuinely a domain concept.

**Coding conventions** (enforced by `CodingConventionRules.java`):
- Constructor injection only — no field injection (use Lombok `@AllArgsConstructor`)
- Controllers and services must be stateless: fields must be `private final` **and of interface type**
- `@Service` classes may not access any class whose simple name ends in `Service` (other than the one they implement); `@RestController` classes may not depend on other `@RestController`s
- Throw specific exceptions from `..service.exception..` — never generic ones
- SLF4J only (Lombok `@Slf4j`, logger field is `LOG`) — no `java.util.logging`, no `System.out`/`System.err`
- No deprecated APIs
- Test classes must live in the same package as the class under test

## Adding a New Feature (API-first)

1. Define the endpoint in `src/main/resources/api/spec/v1/library-api.yaml` **first** — the static spec is the source of truth
2. Add a domain entity (or persistence entity) if needed, plus a Flyway migration
3. Define the interface in `..service..`; implement as `*ServiceImpl` in `..service.impl..`
4. Add the adaptor: `*RestController` (API), `*Controller` (UI), or `*Dao` (persistence). Tag REST controllers with SpringDoc `@Tag`/`@Operation` and register new tags in `ApiConfig`
5. Put DTOs in `..service.dto..`, API-only DTOs in `..adaptors.api.dto..`, exceptions in `..service.exception..`
6. Run the ArchUnit rules early: `./mvnw -Dtest='*Rules' test --offline`
7. Add tests at the right level (below)

## Testing Strategy

| Level | Example | Setup |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xpinjection/test-driven-spring-boot](https://github.com/xpinjection/test-driven-spring-boot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-16 -->
