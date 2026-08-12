---
trigger: always_on
description: mvn install -pl client -am -DskipTests   # must run first; client/ is a Quarkus extension
---

# Qlawkus - Agent Instructions

## Dev Mode Start

```bash
mvn install -pl client -am -DskipTests   # must run first; client/ is a Quarkus extension
mvn -pl app -am quarkus:dev              # from the root: app/ is the only runnable module, and
                                         # capability resolution needs the reactor (see below)
```

Changing `client/` requires re-running `mvn install -pl client -am` then restarting dev mode. Hot reload does NOT apply to extension code.

## Containerized Run

`./run.sh <local|prod|native> [up|build|down|logs|restart|ps]` wraps Docker Compose (`./run.sh --help` for the full list). Unlike dev mode, this needs only Docker: the `app/src/main/docker/Dockerfile.{jvm,native}-build` images are **multi-stage** and compile the whole reactor in-container (build stage = the `ubi9/openjdk-25` builder, which ships Maven; runtime = `ubi9/openjdk-25-runtime`), so there is no host `mvn install` step and no stale-jar trap. Each build Dockerfile has its own `Dockerfile.*-build.dockerignore` so the standard Quarkus `.dockerignore` (for the COPY-target `Dockerfile.jvm`/`.native-micro`) stays intact. The build runs the **full** reactor (`mvn install`), not `-pl app -am`: a fresh local repo needs every `*-deployment` module built so the `extension-descriptor` check on each extension resolves. `local` -> `docker-compose.local.yml`; `prod`/`native` -> `docker-compose.yml` (native via its `native` profile).

## Architecture

Multi-module Maven monorepo with a Quarkus extension pattern (`client/deployment` + `client/runtime`):

- **`composition-model/`** - Thin, **Quarkus-free** module (plain records + Jackson YAML) holding the `agent.yml` composition manifest model: `CompositionManifest`/`BuildTime`/`Posture`, the single `CompositionManifestParser`, the policy+exceptions `isEnabled` logic, and the path conventions (`CompositionPaths`). Shared by the (future) pom generator that reads `build-time` before the build and the running app that reads `runtime` each boot, so the schema and policy never drift (same single-source-of-truth discipline as `KeystoreSecrets`). The `qlawkus.composition.*` location config (`CompositionConfig`, BUILD_TIME) lives in `client`. At runtime `client` also exposes an authenticated composition control plane (`rest.CompositionAdminResource` + `compose.CompositionAdminService`): `POST /api/admin/composition/manifest` validates a new `agent.yml` (via the shared `CompositionManifestParser`) and stages it under `qlawkus.agent.state.root`; `GET /api/admin/composition` returns the active (baked, read from the classpath `agent.yml`) + staged pair; `DELETE .../manifest` discards it. It only records intent (validate + write), never builds - the seam to an external builder is HTTP, not a shared volume, so even a leaked admin credential can at most stage a validated manifest. The same admin surface generalizes to arbitrary config values via `config.metadata.ConfigMetadataIndex` (reads the `quarkus-config-doc` metadata bundled in every extension jar - the same source that generates `config-reference.adoc` - so the property list is never hand-maintained): `RUN_TIME` properties go through `config.RuntimeToggleWriter` + `PUT`/`DELETE /api/admin/runtime-toggles` (writes straight into the `runtime:` override file, applies on restart); `BUILD_TIME`/`BUILD_AND_RUN_TIME_FIXED` properties go through `compose.ConfigOverridesAdminService` + `PUT`/`POST`/`DELETE /api/admin/config-overrides`, staging a `.properties` document that is fetched and promoted into `config-overrides.properties` alongside `agent.yml` by the same redeploy loop - parallel files, never merged, so the pom generator never has to reconcile `application.properties` lines. `secrets.SecretPropertyCatalog` excludes known-secret property names from the index entirely, so a secret is never rendered or writable through either endpoint.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [omatheusmesmo/Qlawkus](https://github.com/omatheusmesmo/Qlawkus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
