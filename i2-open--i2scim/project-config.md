---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

i2scim is a Quarkus-based implementation of the IETF SCIM v2 protocol (RFC 7643/7644). The defining design choice is that **resource types and schema are not hard-coded** — they are loaded at startup from JSON files (`scimSchema.json`, `resourceTypes.json`, `acis.json`) supplied via path, classpath, or K8s ConfigMap. Most code operates on a generic JSON-document model rather than typed resources.

Java 25 / Quarkus 3.34.3 / Jakarta EE 11. Group: `com.independentid`, version: `0.10.0`.

## Build

Three modules, plain Maven from the root:

```bash
mvn install                          # build everything (skips tests by default — see surefire config)
mvn install -DskipTests=false        # build + run tests (requires MongoDB on localhost:27017)
./build.sh -b                        # equivalent to `mvn install` (kept for muscle-memory)
./build.sh -t                        # build + tests
./build.sh --tag <ver>               # build + multi-arch Docker image (load locally)
./build.sh -p --tag <ver>            # build + push to docker.io/independentid
```

Module dependency order: `i2scim-core` → `i2scim-client` → `i2scim-server`. Maven resolves this from the reactor; the explicit `-pl` ordering of the pre-collapse era is no longer required.

The publishable artifacts are `i2scim-core` and `i2scim-client`. `i2scim-server` is the runtime app and sets `<maven.deploy.skip>true</maven.deploy.skip>` to make `mvn deploy` safe (see `docs/publishing.md` for the dormant Maven Central restoration recipe).

## Testing

Unit and integration tests live in `i2scim-server/src/test/java/com/independentid/scim/test/` (organized by area: `memory/`, `mongo/`, `auth/`, `client/`, `http/`, `opa/`, `password/`, `devops/`, `misc/`, `set/`, `ssf/`, `sub/`, `events/`).

```bash
# Whole module:
mvn -pl i2scim-server test

# Single test class or method:
mvn -pl i2scim-server test -Dtest=MemoryProviderTest
mvn -pl i2scim-server test -Dtest=MemoryProviderTest#testCreateUser
```

Test prerequisites:
- **MongoDB** on `localhost:27017` with admin user `admin`/`t0p-Secret`. Override via `TEST_MONGO_URI`, `TEST_MONGO_USER`, `TEST_MONGO_SECRET`. Some tests use Quarkus mongodb devservices and Testcontainers (`org.testcontainers:mongodb`) and need a running Docker daemon.
- **OPA** (optional, for `opa/*` tests) — start via `opa/run-opa.sh`; defaults to `http://localhost:8181/v1/data/i2scim`. Override via `TEST_OPA_URL`.
- **Surefire `argLine`** in the root POM is required: `--add-opens java.base/java.lang=ALL-UNNAMED --add-opens java.base/javax.net.ssl=ALL-UNNAMED --enable-native-access=ALL-UNNAMED`. If you write your own runner, replicate these or SSL-context introspection in the signals code will throw `InaccessibleObjectException` on Java 25.
- **`LoadScimClusterTest` is excluded** from the standard build because it stalls trying to reach a non-existent K8s cluster (see DECISIONS.md, 2026-04-15). The exclusion lives in `i2scim-server/pom.xml` surefire config.

Test-specific configuration is supplied via `QuarkusTestProfile` implementations (e.g. `ScimMemoryTestProfile`) — these set `scim.prov.providerClass`, disable security/events, and remap schema paths to test classpath resources. Follow this pattern; don't put profile-specific values in `application.properties`. Test-only properties that *must* live in `application.properties` use the `%test.` prefix.

## Running locally

- **Dev mode** (memory or mongo, selected by `scim.prov.providerClass` in `application.properties`):
  `mvn -pl i2scim-server quarkus:dev` — endpoint at `http://localhost:8080/`. The `%test.quarkus.mongodb.devservices.enabled=true` profile auto-starts a Mongo container if you flip the provider class to `MongoProvider`.
- **Mongo cluster for SSF/Signals work**: `docker compose -f docker-compose-signals.yml up` (3-node Mongo replica set + 2 SCIM instances) or `docker-compose-mongo-cluster.yml`.

## Architecture

**i2scim-core** — Protocol engine and canonical SCIM schema definitions. Everything else depends on it.
- `protocol/` — request/response types (`RequestCtx`, `ScimResponse`, `ListResponse`), filter parsing (`Filter`, `LogicFilter`, `ValuePathFilter`, `AttributeFilter`), JSON Patch.
- `schema/` — `SchemaManager` loads JSON definitions and exposes attribute metadata; `Schema`, `Attribute`, `ResourceType`.
- `resource/` — generic `ScimResource` / `Value` model that everything serializes through.
- `op/` — operation objects (`CreateOp`, `GetOp`, `PatchOp`, `BulkOps`...) executed by a worker pool.
- `core/` — `ConfigMgr` (config singleton), `PoolManager` (worker threads).
- `backend/` — `IScimProvider` SPI; provider implementations live in `i2scim-server` (memory + mongo).
- `plugin/` — `IScimPlugin` SPI for pre/post-transaction hooks; `PluginHandler` dispatches.
- `events/` — `IEventHandler` SPI for async event publishing.
- `security/` — ACI parsing/evaluation (`AccessControl`, `AciSet`, `AccessManager`); built-in `ScimAuthMechanism`, `ScimBasicIdentityProvider`. Auth is per-request (no sessions/cookies).
- `pwd/` — PBKDF2 password handling (FIPS-aligned).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [i2-open/i2scim](https://github.com/i2-open/i2scim) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
