---
trigger: always_on
description: Kiit is an open-source Kotlin Multiplatform (KMP) framework developed and maintained by CodeHelix. It is a monorepo of modular libraries organized into categories, designed for sharing code across server (JVM/Kotlin), Android, JavaScript/TypeScript, and iOS projects.
---

# Kiit Framework — Claude Code Context

## About Kiit

Kiit is an open-source Kotlin Multiplatform (KMP) framework developed and maintained by CodeHelix. It is a monorepo of modular libraries organized into categories, designed for sharing code across server (JVM/Kotlin), Android, JavaScript/TypeScript, and iOS projects.

- **Domain**  : kiit.dev
- **GitHub**  : https://github.com/slatekit/kiit
- **Organization**: CodeHelix

---

## Definition Files — Single Source of Truth

The complete registry of categories, libraries, platform targets, migration state, and metadata is defined in the `./definition/` folder at the repo root.

```

./definition/
├── kiit-framework-definition.json    ← root: org, enums, category registry
└── categories/
    ├── core.json
    ├── parse.json
    ├── app.json
    ├── infra.json
    ├── resilience.json
    ├── data.json
    ├── services.json
    ├── vendor.json
    ├── connect.json
    └── internal.json
```

**Do not add, rename, or remove libraries or categories in this document.**
Make all structural changes in the definition files. This document contains
rules and conventions only — the definition files contain the registry.

All skills read from these files. When performing any task involving a specific
library or category, always read the relevant category JSON first.

### Target resolution rule

A library's effective platform targets are resolved as follows:
- If `library.targets` is present → use `library.targets`
- If `library.targets` is absent  → inherit `category.defaultTargets`

Skills and scripts must always apply this rule. Never assume one or the other
is always present.

---

## Current State of the Framework

The framework is currently in a pre-migration state. All libraries exist but do not yet
follow the naming conventions, structure, or build configuration described in this document.

| Concern                | Current State                                                  |
|------------------------|----------------------------------------------------------------|
| Gradle DSL             | Groovy (`build.gradle`)                                        |
| Kotlin version         | 1.9.x                                                          |
| Ktor version           | 2.x.x (tied to Kotlin 1.8.x)                                   |
| Gradle version         | 8.x                                                            |
| Version catalog        | Not in use — versions hardcoded in build files                 |
| Multiplatform (KMP)    | Not configured — JVM only                                      |
| Folder structure       | Does not match desired category structure                      |
| Artifact IDs           | Prefixed with `kiit-{library}` not `kiit-{category}-{library}` |
| Package names          | Rooted at `kiit.*` not `kiit.{category}.{library}*`            |
| Publishing             | GitHub Packages only                                           |
| ktlint                 | Not configured                                                 |
| Android Gradle Plugin  | 7.x                                                            |

The `migration` block in each library's category JSON captures the current artifact ID,
current package name, current folder location, and migration status for every library.

---

## Desired / Future State of the Framework

| Concern                | Desired State                                               |
|------------------------|-------------------------------------------------------------|
| Gradle DSL             | Kotlin (`build.gradle.kts`)                                 |
| Kotlin version         | 2.3.21                                                      |
| Ktor version           | 3.5.0                                                       |
| Gradle version         | 8.13                                                        |
| Version catalog        | `gradle/libs.versions.toml` — single source for versions    |
| Multiplatform (KMP)    | Configured per platform target matrix in definition files   |
| Folder structure       | Matches category structure under `src/{category}/{library}` |
| Artifact IDs           | Follow `kiit-{category}-{library}` convention               |
| Package names          | Rooted at `kiit.*` following naming conventions             |
| Publishing             | Maven Central (stable), GitHub Packages (pre-release)       |
| ktlint                 | Configured and enforced across all modules                  |
| Android Gradle Plugin  | 8.7.0                                                       |

Versions are managed centrally in `gradle/libs.versions.toml`. No module-level
`build.gradle.kts` should hardcode a library version — always reference the catalog.

---

## Naming Conventions

### Maven / Gradle

| Element      | Rule                                                                                   | Example             |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [slatekit/kiit](https://github.com/slatekit/kiit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
