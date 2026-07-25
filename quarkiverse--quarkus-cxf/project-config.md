---
trigger: always_on
description: Quarkus CXF is a [Quarkus](https://quarkus.io/) extension for developing SOAP web services and clients using [Apache CXF](https://cxf.apache.org/). It follows the Quarkus extension model (runtime + deployment split) and supports both JVM and GraalVM native compilation.
---

# AGENTS.md — Quarkus CXF

## Project overview

Quarkus CXF is a [Quarkus](https://quarkus.io/) extension for developing SOAP web services and clients using [Apache CXF](https://cxf.apache.org/). It follows the Quarkus extension model (runtime + deployment split) and supports both JVM and GraalVM native compilation.

- **Repository**: https://github.com/quarkiverse/quarkus-cxf
- **Documentation**: https://docs.quarkiverse.io/quarkus-cxf/dev/
- **License**: Apache 2.0
- **Baseline Java version**: see `maven.compiler.release` property in root `pom.xml`
- **Primary dependencies**: Quarkus (see `quarkus.version` in root `pom.xml`) and Apache CXF (see `cxf.version`)

## Directory structure

```
├── bom/                       # Bill of Materials for dependency management
├── bom-test/                  # Test BOM
├── docs/                      # Antora documentation (AsciiDoc)
├── extensions/                # Quarkus extensions
├── integration-tests/         # Integration test modules (client, server, mTLS, WS-*, etc.)
├── perf-tests/                # Performance benchmarks (Hyperfoil)
├── test-util-parent/          # Shared test utilities
├── .github/                   # GitHub Actions workflows and actions
└── .claude/                   # Claude Code settings and skills
```

## Build system

Maven multi-module project. Use `./mvnw` from the repository root or `mvnd` if available.

### Key commands

```sh
# Build everything (skip tests)
./mvnw clean install -DskipTests -Dquarkus.build.skip

# Build and test a single integration test module
./mvnw clean verify -f integration-tests/client-server

# Run native tests (requires GraalVM)
./mvnw clean verify -f integration-tests/client-server -Pnative -Dquarkus.native.container-build

# Sync dependency versions annotated with @sync
./mvnw cq:sync-versions -N
```

### Code formatting

The project enforces formatting via Maven plugins — `formatter-maven-plugin` (Java sources), 
`impsort-maven-plugin` (import order), and `sortpom-maven-plugin` (POM file ordering). 
Generated files under `**/generated/*.java` are excluded.
The plugins a part of Maven lifecycle, so there is no need to call them separately, if Maven compile phase has passed
by calling `./mvnw clean compile ...`, `./mvnw clean test ...`, `./mvnw clean verify ...` or `./mvnw clean install ...`.

POM dependencies are sorted by: scope, type, groupId, artifactId.

## Extension architecture

Each extension follows the Quarkus convention:

- **Runtime module** (`runtime/`): Application-facing code, configuration annotations, CDI producers
- **Deployment module** (`deployment/`): Build-time processors, build items, GraalVM substitutions
- **Metadata**: `META-INF/quarkus-extension.yaml`

The core extension (`extensions/core/`) additionally contains:
- `vertx-client/` — Vert.x-based HTTP client for CXF
- `axiom-api-stub/` — Stub for Axiom API to avoid pulling the full dependency

## Configuration

SOAP endpoints and clients are configured via `application.properties` using the `quarkus.cxf.*` namespace:

```properties
# Expose a SOAP endpoint
quarkus.cxf.endpoint."/hello".implementor = com.example.HelloServiceImpl

# Configure a SOAP client
quarkus.cxf.client.myClient.client-endpoint-url = http://localhost:8080/soap/hello
quarkus.cxf.client.myClient.service-interface = com.example.HelloService

# WSDL-to-Java code generation
quarkus.cxf.codegen.wsdl2java.my-codegen.includes = wsdl/hello.wsdl
```

## Testing

- **JVM tests**: Standard Maven Surefire/Failsafe tests on Java versions specified in `.github/workflows/build.yml`
- **Native tests**: GraalVM native image tests activated by `-Pnative -Dquarkus.native.container-build`
- **Integration tests**: Each module in `integration-tests/` tests a specific feature area
- **Performance tests**: Hyperfoil benchmarks in `perf-tests/`
- **Test utilities**: Shared helpers in `test-util-parent/test-util/` and `test-util-parent/test-util-internal/`

## CI/CD

GitHub Actions workflows in `.github/workflows/`:

| Workflow | Purpose |
|----------|---------|
| `build.yml` | Main CI: builds and tests on multiple JDKs and platforms |
| `codeql.yml` | CodeQL security analysis |
| `pre-release.yml` | Pre-release checks |
| `release-prepare.yml` | Prepare a release (update versions) |
| `release-perform.yml` | Perform the release (deploy artifacts) |
| `rebase-integration-branch.yml` | Maintain integration branches for CXF/Quarkus as built from their `main` and LTS branches |

Release versions are managed in `.github/project.yml`.

## Documentation

Documentation is written in AsciiDoc using the Antora framework, located in `docs/modules/ROOT/pages/`. Key sections:

- `user-guide/` — Getting started, SOAP client/service development, SSL/TLS, logging, JAXB
- `reference/extensions/` — Per-extension configuration reference (auto-generated by `cq-maven-plugin`)
- `contributor-guide/` — How to contribute and release
- `security-guide/` — WS-Security and WS-SecurityPolicy
- `release-notes/` — Per-version release notes

## Coding conventions


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [quarkiverse/quarkus-cxf](https://github.com/quarkiverse/quarkus-cxf) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->
