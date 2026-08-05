---
trigger: always_on
description: Nuxeo is an Enterprise Content Management (ECM) platform. This is the `nuxeo-lts` monorepo (version `2025.x-SNAPSHOT`), a massive Maven multi-module Java project hosted at `github.com/nuxeo/nuxeo-lts` on branch `2025`.
---

# Nuxeo LTS 2025 - Agent Development Guide

## Project Overview

Nuxeo is an Enterprise Content Management (ECM) platform. This is the `nuxeo-lts` monorepo (version `2025.x-SNAPSHOT`), a massive Maven multi-module Java project hosted at `github.com/nuxeo/nuxeo-lts` on branch `2025`.

### Module Hierarchy

```
nuxeo-ecm (root)
├── boms/                    # Bill of Materials (OpenSearch 1.x BOM)
├── modules/
│   ├── runtime/             # ~27 modules - Core runtime, OSGi, streams, Kafka, MongoDB driver, KV store
│   ├── core/                # ~29 modules - Document model, storage engines (SQL, MongoDB, DBS, mem), bulk ops, binary managers
│   └── platform/            # ~96 modules - High-level services: REST API, audit, auth, automation, workflows, Drive, CMIS
├── parent/                  # Re-exportable parent POM for out-of-tree builds (addons, customer projects)
├── server/                  # Server distribution (Tomcat-based launcher, NXR assembly)
├── packages/                # ~43 Nuxeo marketplace packages
├── ftests/                  # ~35 functional test modules (tiered: Tier5, Tier6, Tier7)
├── docker/                  # Docker images (nuxeo, nuxeo-benchmark)
└── ci/                      # Jenkins pipelines, Helm charts, CI scripts
```

### POM Hierarchy

```
nuxeo-ecm (root, org.nuxeo)             # All dependency versions managed here
├── nuxeo-parent (org.nuxeo)             # Re-exportable parent for external projects
└── nuxeo-modules (org.nuxeo)
      ├── nuxeo-runtime-parent           # Parent for all runtime modules
      ├── nuxeo-core-parent              # Parent for all core modules
      └── nuxeo-platform-parent          # Parent for all platform modules
            └── nuxeo-*-parent           # Some platform areas have sub-parents (e.g., audit, webengine)
```

Module POMs declare dependencies **without `<version>` tags** - all versions are centrally managed in the root POM.

### Build System

- **Maven** multi-module with root POM managing all dependency versions (~5400 lines)
- **Java 21** (Azul Zulu JDK) with `<release>21</release>`
- **Maven 3.6.3+** required
- Default build: `mvn install` (builds `boms` + `modules` only)
- Full build: `mvn install -Pdistrib,docker -DskipTests -T6`
- Code formatting: `mvn spotless:check` / `mvn spotless:apply` (Eclipse formatter, ratcheted from `origin/2025`)
- MAVEN_OPTS: `-Xmx4g -Xms2g`

## Languages

- **Java** for the main Nuxeo application code (99%+ of the codebase)
- **XML** for component declarations, extension contributions, schemas (XSD), and Maven POMs
- **FreeMarker** (`.ftl`) for server-side templates: email notifications and WebEngine page rendering
- **JavaScript** legacy only: vendored jQuery/TinyMCE in WebEngine pages (no modern frontend framework, no npm/node)
- **Groovy** for Jenkins CI pipelines and server admin scripts
- **Python** for release tooling and test infrastructure (not part of the application)
- **Shell** for Docker entrypoints and server management scripts

## Java Conventions

### Modern Java Features (USE THEM)

- **`var`**: Use local variable type inference wherever the type is obvious from context
- **Records**: Use for immutable value types and DTOs
- **Sealed interfaces/classes**: Use when appropriate for closed type hierarchies
- **Pattern matching `instanceof`**: Use `if (value instanceof String s)` instead of cast-after-check
- **Switch expressions**: Use `var x = switch (y) { case ... -> ...; };` for exhaustive switches
- **Text blocks**: Use `"""` for multi-line strings
- **`String.formatted()`**: Prefer `"text %s".formatted(value)` over `String.format()`
- **Streams**: Use `.stream()`, `.map()`, `.filter()`, `.toList()` for collection transformations
- **`List.copyOf()`, `Map.copyOf()`**: Use in record constructors for defensive copying
- **Try-with-resources**: Always use for `AutoCloseable` resources

### Code Style

- **4-space indentation** (no tabs)
- **K&R braces** (opening brace on same line)
- **~120 character line width**
- **Formatting**: Enforced by Spotless plugin with Eclipse formatter. Run `mvn spotless:apply` to auto-format
- **No wildcard imports** (enforced by Spotless)
- **Null annotations**: Use `jakarta.annotation.Nonnull` (and `jakarta.annotation.Nullable`) for null-safety contracts in method signatures. Do NOT use `jakarta.validation.constraints.NotNull` for this purpose -- that annotation is reserved for Bean Validation (runtime constraint checking), not API contracts
- **`@SuppressWarnings` conventions**:
  - `"unchecked"` for generic type casts (most common)
  - `"resource"` for `AutoCloseable` resources managed elsewhere (always add an inline comment explaining why)
  - `"deprecation"` / `"removal"` for intentional calls to deprecated APIs (e.g., backward-compatible bridges)

### Package Naming

```
org.nuxeo.runtime.*          # Runtime-layer modules
org.nuxeo.ecm.core.*         # Core-layer modules (document model, storage, events)
org.nuxeo.ecm.platform.*     # Platform-layer modules (high-level services)
org.nuxeo.common.*            # Common utilities (xmap, Environment, etc.)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nuxeo/nuxeo](https://github.com/nuxeo/nuxeo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->
