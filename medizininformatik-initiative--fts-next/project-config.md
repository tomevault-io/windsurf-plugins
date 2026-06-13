---
trigger: always_on
description: This file provides guidance to coding agents when working with code in this repository.
---

# AGENTS.md

This file provides guidance to coding agents when working with code in this repository.

## Build Commands
- **Build project**: `mvn clean install` or `make build`
- **Run tests**: `mvn clean verify` or `make test`
- **Run a single test**: `mvn test -Dtest=TestClassName#testMethodName`
- **Run a specific test class**: `mvn test -Dtest=TestClassName`
- **Run integration tests**: `mvn failsafe:integration-test`
- **Build coverage report**: `mvn jacoco:report-aggregate@report` or `make coverage`
- **Lint/style check**: `mvn checkstyle:check`
- **Build specific agent**: `make clinical-domain-agent` (or other agent name)
- **Build Docker images**: `make all` (builds all agents as containers)
- **Run specific agent module**: `mvn clean package -DskipTests --projects clinical-domain-agent --also-make`

## Architecture Overview

FTS-next is a multi-agent healthcare data transfer system implementing the SMITH architecture for secure FHIR data exchange between clinical and research domains.

### Core Architecture Pattern

The system follows a **three-agent architecture** with separation of concerns:

- **Clinical Domain Agent (CDA)**: Extracts and processes clinical data, applies initial deidentification
- **Trust Center Agent (TCA)**: Manages consent verification and pseudonymization services
- **Research Domain Agent (RDA)**: Receives and stores deidentified data in research systems

### Module Structure

- **api/**: Core interfaces and data models (`TransferProcessStep`, `ConsentedPatient`, `TransportBundle`)
- **util/**: Shared utilities (authentication, FHIR codecs, WebClient factories, metrics)
- **test-util/**: Testing infrastructure (`AbstractAuthIT`, `AbstractConnectionScenarioIT`, test data generators)
- **{agent}-agent/**: Agent-specific implementations following the same pattern:
  - Transfer process orchestration (`DefaultTransferProcessRunner`)
  - Step implementations (`impl/` package)
  - REST controllers (`rest/` package)
  - Configuration classes
- **trust-center-agent/adapters/**: Pluggable backends for pseudonymization (gPAS, Vfps, entici)

### Plugin Architecture

The system uses a factory pattern for pluggable processing steps:

```java
public interface TransferProcessStepFactory<STEPTYPE, CCONF, ICONF> {
  Class<ICONF> getConfigType();
  STEPTYPE create(CCONF commonConfig, ICONF implConfig);
}
```

Key step types:
- **CohortSelector**: Identifies patients based on consent (TcaCohortSelector, FhirCohortSelector)
- **DataSelector**: Extracts patient data (EverythingDataSelector)
- **Deidentificator**: Removes/transforms identifying information (DeidentifhirStep, FhirPseudonymizerStep)
- **BundleSender**: Transmits data between agents (RdaBundleSender, FhirStoreBundleSender)

### Configuration System

Transfer processes are defined in YAML project files under `projects/` directories:

```yaml
cohortSelector:
  tca:
    policies: ["research-policy"]
    domain: "clinical"
dataSelector:
  everything:
    maxPageSize: 1000
deidentificator:
  deidentifhir:
    profile: "research-profile"
bundleSender:
  rda:
    endpoint: "https://rda.example.com"
```

### Technology Stack

- **Java 21**
- **Spring Boot 4.0.x** with WebFlux (reactive programming)
- **HAPI FHIR 8.10.x** for FHIR R4 processing
- **Project Reactor** for non-blocking operations
- **Redisson 4.5.0** for reactive Redis operations (RMapCacheReactive for TTL-based session storage)
- **Maven** for build management
- **Docker** for containerization
- **JUnit 6** with AssertJ for testing

## Authentication & Security

The system supports multiple authentication methods configurable per agent:

- **None**: Development/testing
- **Basic Auth**: Username/password
- **OAuth2**: Client credentials flow
- **Client Certificates**: Mutual TLS

Authentication is configured via `HttpServerAuthConfig` with per-endpoint settings.

## Testing Strategy

- **Unit Tests**: Standard JUnit tests for business logic
- **Integration Tests**: Spring Boot tests with `@SpringBootTest`
- **Connection Scenario Tests**: Resilience testing using `AbstractConnectionScenarioIT`
- **End-to-End Tests**: Full workflow testing across agents (`.github/test/`)

For integration tests, extend `AbstractAuthIT` to ensure proper authentication handling.

**Note**: `OAuth2AuthIT` test errors are expected in local development environments without proper OAuth2 infrastructure.

## Code Style Guidelines
- Follow Google Java Style Guide (enforced by checkstyle with google_checks.xml)
- Use standard Java imports ordering (no wildcards)
- Prefer static imports for utility methods (e.g., `import static java.util.Objects.requireNonNull;`)
- Fail fast on null: use `@NotNull` annotations + `requireNonNull()` at boundaries, don't defensively check null throughout
- Avoid unnecessary comments: code should be self-explanatory; only comment non-obvious "why", not "what"
- Use lombok annotations for boilerplate reduction (e.g., @Slf4j, @ToString), but avoid `@UtilityClass` — use interfaces with static methods instead
- Follow standard Java naming conventions (camelCase for methods/variables, PascalCase for classes)
- Use records for immutable data classes where appropriate
- Proper exception handling with descriptive messages

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [medizininformatik-initiative/fts-next](https://github.com/medizininformatik-initiative/fts-next) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-12 -->
