---
trigger: always_on
description: This is a Spring Boot starter that extracts Kogito Serverless Workflow custom URI scheme infrastructure into three publishable Gradle modules plus a sample app. It enables consuming applications to get full `dmn://`, `anax://`, and `map://` custom function support by adding a single Gradle dependency and plugin. Consumers never reference `org.kie.kogito` artifacts directly.
---

# Custom Workflow Spring Boot Starter

## Project Overview

This is a Spring Boot starter that extracts Kogito Serverless Workflow custom URI scheme infrastructure into three publishable Gradle modules plus a sample app. It enables consuming applications to get full `dmn://`, `anax://`, and `map://` custom function support by adding a single Gradle dependency and plugin. Consumers never reference `org.kie.kogito` artifacts directly.

Governance assets (DMN models, Jolt mapping specs) are authored via Copilot + MCP tools connected to the [Metadata Management Platform](docs/canonical-metadata-server.md), committed to `src/main/resources/`, and read locally at build time. Builds are fully self-contained with zero external service dependencies. At runtime, the starter registers the catalog with the metadata server for observability and drift detection.

## Tech Stack

- **Java 17** — minimum target, no Java 21+ features
- **Spring Boot 3.3.7** — auto-configuration uses `META-INF/spring/AutoConfiguration.imports` (not deprecated `spring.factories`)
- **Gradle 8.10+** — Gradle-only, no Maven support
- **Kogito 10.1.0** — pinned version, managed centrally via `kogitoVersion` property
- **CNCF Serverless Workflow spec v0.8**

## Architecture: Governance Asset Lifecycle

The project follows a three-phase lifecycle for governance assets:

### Phase 1: Author (IDE → Metadata Server)
- **Primary:** Copilot + MCP tools — `create_decision`, `update_decision`, `validate_decision`, etc. (19 validated MCP tools)
- **Fallback:** Metadata server REST API / UI for non-Copilot workflows
- Copilot writes DMN/Jolt files directly into `src/main/resources/`
- Developer commits governance assets to git
- All MCP write tools force `status: "draft"` — human promotes to `active` via metadata platform UI

### Phase 2: Build (Local Only)
- `FunctionTypeHandler` SPI implementations (in `anax-kogito-codegen-extensions`) are discovered via `ServiceLoader` during Kogito codegen
- They parse custom URIs and emit `WorkItemNode` entries in the generated process code
- Reads `.sw.json` and `.dmn` files from `src/main/resources/` — **no metadata server dependency**
- Builds are fully self-contained and reproducible

### Phase 3: Runtime (Spring Boot)
- `DefaultKogitoWorkItemHandler` subclasses (in `anax-kogito-spring-boot-starter`) are registered by name with `WorkItemHandlerConfig`
- When the process engine reaches a `WorkItemNode`, it dispatches to the handler matching the `workName`
- On `ApplicationReadyEvent`, the starter publishes the catalog to the metadata server (`POST /api/registrations`)
- Registration enables observability, drift detection, and service catalog features
- Registration failure is logged but does NOT prevent the application from starting

**Critical invariant:** The `workName(scheme)` set during codegen must exactly match the name used in `register(scheme, handler)` at runtime.

## Module Structure

| Module | Purpose | Phase |
|--------|---------|-------|
| `anax-kogito-codegen-extensions` | SPI `FunctionTypeHandler` implementations for `dmn://`, `anax://`, `map://` | Build time |
| `anax-kogito-spring-boot-starter` | Auto-configuration: `WorkItemHandler` beans + `WorkItemHandlerConfig` + metadata catalog REST endpoint + runtime registration with metadata server | Runtime |
| `anax-kogito-codegen-plugin` | Gradle plugin: `generateKogitoSources` task, classpath wiring, BOM management, `catalog.json` generation | Build time |
| `anax-kogito-sample` | Example Spring Boot app demonstrating all features (not published) | Both |

## Custom URI Schemes

All custom functions use `"type": "custom"` in `.sw.json` definitions:

| Scheme | URI Pattern | Purpose | Handler |
|--------|-------------|---------|---------|
| `dmn://` | `dmn://{namespace}/{modelName}` | Evaluate a DMN decision model in-process | `DmnWorkItemHandler` |
| `anax://` | `anax://{beanName}/{methodName}` | Invoke a Spring bean method (default method: `execute`) | `AnaxWorkItemHandler` |
| `map://` | `map://{mappingName}` | Apply a Jolt data transformation (spec committed in `src/main/resources/META-INF/anax/mappings/`) | `MapWorkItemHandler` |

### Bean method contract for `anax://`

```java
public Map<String, Object> methodName(Map<String, Object> params)
```

### Jolt transformation contract for `map://`

Jolt specs are committed at `src/main/resources/META-INF/anax/mappings/{mappingName}.json`. At runtime, `MapWorkItemHandler` loads the spec and applies the Jolt transformation. The initial implementation is a stub (Jolt engine wired in a later iteration).

## Kogito Source Code References

When researching Kogito internals, use the correct branch:

| What | Branch / URL |
|------|-------------|
| **Gradle plugin for Kogito codegen** (does not exist in 10.1.0) | `main` branch: https://github.com/apache/incubator-kie-kogito-runtimes/tree/main |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/margic) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
