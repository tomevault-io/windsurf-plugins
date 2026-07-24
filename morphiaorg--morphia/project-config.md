---
trigger: always_on
description: Morphia is a Java ODM (Object-Document Mapper) for MongoDB.
---

# Morphia – Copilot Coding-Agent Instructions

## Project overview

Morphia is a Java ODM (Object-Document Mapper) for MongoDB.
It maps annotated Java (and Kotlin) POJOs to/from BSON documents.
The current development branch targets **Java 17**, **Maven 4**, and the **MongoDB Java Driver 5.x**.

---

## Repository layout

| Directory | Purpose |
|-----------|---------|
| `core/` | Main library – mapping, query, aggregation, config |
| `kotlin/` | Kotlin extensions for Morphia |
| `critter/` | Annotation-processor / compile-time query-criteria generator |
| `validation/` | Bean-Validation integration |
| `rewrite/` | OpenRewrite recipes for migrating user code |
| `build-plugins/` | Custom Maven plug-ins used during the build |
| `util/` | Shared test utilities and internal helpers |
| `examples/` | Runnable usage examples (also serve as integration tests) |
| `docs/` | Antora-based documentation sources |
| `audits/` | Code-audit tooling (enabled via the `audits` Maven profile) |
| `config/` | Shared IDE formatter / FindBugs / RevAPI config files |
| `.github/` | CI workflows, issue templates, JBang helper scripts |

---

## Toolchain

- **Java 17** (Temurin) — enforced in `.sdkmanrc` and `pom.xml`
- **Maven 4** (wrapper at `./mvnw`) — use the wrapper, not a system Maven
- **Kotlin** — compiled alongside Java in the `kotlin/` subproject
- **JUnit** — test framework 
- **Testcontainers** — spins up a real `mongo:<major>` Docker container for tests
- **SmallRye Config** — drives `MorphiaConfig` via `morphia-config.properties` on the classpath

---

## Building

```bash
# Compile everything, skip tests
./mvnw install -DskipTests

# Compile and run all tests (requires Docker for Testcontainers)
./mvnw verify

# Build a single subproject
./mvnw install -DskipTests -pl core

# Run tests against a specific MongoDB version (default is 8.0.0)
./mvnw surefire:test -Dmongodb=7.0.0

# Run tests against a specific driver version
./mvnw surefire:test -Ddriver.version=5.3.0
```

Use `test-all.sh` for an interactive matrix test across multiple server / driver combinations.

---

## Testing

### Framework and conventions

- **JUnit** is used
- Tests that need a running MongoDB instance extend `TestBase` (in `core/src/test/java/dev/morphia/test/`).
- `MorphiaTestSetup` (parent of `TestBase`) starts a Testcontainers `MongoDBContainer` in `@BeforeSuite`; tests share a single container per suite run.
- Pass `-Dmongodb=local` to point at a local `mongod` on `27017` instead of a container.
- The test database name is `morphia_test` (constant `TestBase.TEST_DB_NAME`).

### Running tests

```bash
# All tests in a module
./mvnw test -pl core

# A single test class
./mvnw test -pl core -Dtest=TestDatastore

# A single test method
./mvnw test -pl core -Dtest=TestDatastore#testSave

# Skip tests but build everything
./mvnw install -DskipTests
```

### Test configuration

Create `src/test/resources/morphia-config.properties` (or supply a `MorphiaConfig` programmatically) to override defaults in tests.

---

## Key source-code entry points

| Class | Role |
|-------|------|
| `dev.morphia.Morphia` | Factory – call `Morphia.createDatastore(mongoClient)` or `Morphia.createDatastore(mongoClient, config)` |
| `dev.morphia.Datastore` | Primary API for CRUD, queries, and aggregations |
| `dev.morphia.MorphiaDatastore` | Concrete `Datastore` implementation |
| `dev.morphia.config.MorphiaConfig` | SmallRye Config-backed configuration interface (prefix `morphia`) |
| `dev.morphia.mapping.Mapper` | Maps Java types ↔ BSON documents |
| `dev.morphia.annotations.*` | `@Entity`, `@Id`, `@Property`, `@Indexed`, `@Version`, etc. |

---

## Annotations

Entities must be annotated with `@Entity`.
The identifier field must be annotated with `@Id`.
Additional mapping annotations live in `dev.morphia.annotations`.

Example:

```java
@Entity("users")
public class User {
    @Id ObjectId id;
    String name;
    int age;
}
```

---

## Configuration

Morphia reads `morphia-config.properties` from the classpath (via SmallRye Config).
All keys are prefixed with `morphia.`.
Common properties:

| Key | Default | Description |
|-----|---------|-------------|
| `morphia.database` | *(required)* | MongoDB database name |
| `morphia.collection-naming` | `identity` | Collection naming strategy |
| `morphia.property-naming` | `identity` | Property / field naming strategy |
| `morphia.discriminator` | `className` | Discriminator function for polymorphism |
| `morphia.auto-import-models` | `true` | Auto-discover `@Entity` classes |

Programmatic configuration uses the builder returned by `MorphiaConfig`:

```java
MorphiaConfig config = MorphiaConfig.load(); // from classpath
// or build manually with ManualMorphiaConfig
```

---

## CI / GitHub Actions

Workflows live in `.github/workflows/`:

| File | Trigger | What it does |
|------|---------|-------------|
| `pull-request.yml` | PR opened/updated | Code audits + full build |
| `build.yml` | Push to non-dependabot branches / tags | Matrix build across MongoDB and driver versions; release |
| `audits.yml` | Scheduled / manual | Extra code-quality audits |
| `claude.yml` | `@claude` comment | Invokes Claude AI assistant |


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MorphiaOrg/morphia](https://github.com/MorphiaOrg/morphia) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
