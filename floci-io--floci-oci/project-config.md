---
trigger: always_on
description: Guidance for AI coding agents working in the floci-oci repository.
---

# Agent Guide — floci-oci

Guidance for AI coding agents working in the floci-oci repository.

## Project Overview

floci-oci is a Java-based local **Oracle Cloud Infrastructure (OCI)** emulator built on Quarkus.
Its goal is full OCI SDK and OCI CLI compatibility through real OCI wire protocols, not
convenience APIs. It is the OCI sibling of floci (AWS, port 4566), floci-az (Azure, 4577)
and floci-gcp (GCP, 4588).

- Port: **4599**
- Stack: Java 25, Quarkus, JUnit 5, RestAssured, Jackson
- Package root: `io.floci.oci`
- Config prefix: `floci-oci.*` / env `FLOCI_OCI_*`

## First Principles

1. Preserve OCI protocol compatibility
2. Match OCI SDK and CLI behavior
3. Reuse existing patterns
4. Prefer correctness over convenience
5. Keep changes narrow and testable

Critical rules:

- Do not introduce custom endpoint shapes
- Do not change request or response formats for convenience
- Never invent protocol behavior — consult the reference sources under `local/oracle/`
  (see "OCI Source as Reference" below; `make refs` downloads them)

## Architecture

Layered: **Controller** (JAX-RS, parses OCI REST input) → **Service** (business logic, throws
`OciException`) → **Model** (`model/Stored*.java`, `@RegisterForReflection`).

Core infrastructure (`io.floci.oci.*`):

- `config/EmulatorConfig` — single `@ConfigMapping(prefix = "floci-oci")` interface
- `core/common/` — `OciException` + `OciExceptionMapper` (error shape
  `{"code":"...","message":"..."}` + `opc-request-id` header), `ServiceRegistry` +
  `ServiceDescriptor` (self-registering), `ServiceEnabledFilter` (503 for disabled services),
  `RequestContext` (tenancy/user/region)
- `core/storage/` — `StorageBackend` (memory/persistent/hybrid/wal via `StorageFactory`),
  `TenancyAwareStorageBackend` (keys prefixed by tenancy OCID)
- `core/common/docker/` — sidecar container management
- `lifecycle/` — `EmulatorLifecycle`, init hooks, `/health` + `/_floci-oci/*` endpoints

## OCI Protocol Rules

- Every service except Object Storage uses a date-versioned path prefix
  (Identity `/20160918/…`); Object Storage uses `/n/{namespace}/b/{bucket}/o/{object}`.
  JAX-RS `@Path` matching dispatches directly — there is no routing filter.
- Errors: `{"code": "...", "message": "..."}` body + correct HTTP status. 404 is
  `NotAuthorizedOrNotFound` (OCI deliberately conflates the two).
- Every response carries an `opc-request-id` header.
- Pagination: `limit`/`page` query params in, `opc-next-page` response header out.
  Some list APIs return a bare JSON array — verify each against the SDK model.
- OCIDs: `ocid1.<type>.<realm>.<region>.<unique>` (region omitted for global resources).
- Auth: the `Authorization: Signature …` header is parsed for tenancy/user context only;
  the RSA signature is never verified.
- **Tenancy is the storage partition; compartment is a field on each resource** filtered
  via `?compartmentId=`. Do not conflate them.
- Async operations return 202 + `opc-work-request-id` and are polled via work requests.

## Registration Pattern (no service-keyed switches)

Each service registers itself in an `@Observes StartupEvent` method:

```java
void onStart(@Observes StartupEvent ev) {
    serviceRegistry.register(ServiceDescriptor.builder("objectstorage")
            .enabled(config.services().objectstorage().enabled())
            .storageKey("objectstorage")
            .resourceClasses(ObjectStorageController.class)
            .build());
}
```

`ServiceRegistry`, `ServiceEnabledFilter`, `StorageFactory` and the banner resolve service
metadata through descriptors. Adding a service must never require editing a switch in core.

## Services with Container Sidecars

Some services launch real Docker containers (sidecars). **`services/functions/` is the
reference implementation** — copy its shape:

1. **One `mock()` flag is the only container toggle** on the service's config
   (`@WithDefault("false")`, env `FLOCI_OCI_SERVICES_<SVC>_MOCK`). No separate opt-in.
   `src/test/resources/application.yml` always sets `mock: true` so the suite never
   starts containers.
2. **The Manager (driver) is flag-free** and owns only mechanics: lazy idempotent
   `ensureStarted()` (self-healing via `isContainerRunning`), a single cheap
   `boolean isReady()` probe, `stop()`. It goes through `ContainerBuilder` /
   `ContainerLifecycleManager` — never raw `dockerClient` calls — and MUST
   `portAllocator.release(port)` in the stop path (leaked ports exhaust the range).
3. **The service owns the gate**: every container interaction sits behind `!mock()`;
   mock mode keeps the management plane fully usable with synthetic data-plane results.
4. **Never block a request thread on readiness** — poll asynchronously or bound the wait
   to the data-plane call that actually needs the sidecar (Functions bounds it to invoke).
5. **Teardown**: `@PreDestroy` stops the sidecar, and the service implements
   `Resettable` so `POST /_floci-oci/state/reset` also removes containers/volumes.
6. **Tests**: the standard trio runs in mock mode; add a `<Svc>DockerTest` with
   `@TestProfile` flipping `mock=false`, `assumeTrue(docker socket)` in `@BeforeAll`,
   `PER_CLASS` + ordered methods, and a final cleanup test (not `@AfterAll` — the
   server port is gone by then).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [floci-io/floci-oci](https://github.com/floci-io/floci-oci) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-10 -->
