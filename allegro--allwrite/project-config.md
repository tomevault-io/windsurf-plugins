---
trigger: always_on
description: - Never write any comments in code
---

# Custom Rules

- Never write any comments in code
- Exception: tests must use `// given`, `// when`, `// then`, `// expect`, and `// and` structural comments to separate test phases (does NOT apply to recipe
  tests using `before`/`beforeAndAfter`/`after` RewriteTest patterns)
- Update AGENTS.md every time project structure is changed

---

# Project Overview

**allwrite** is a CLI tool and collection of [OpenRewrite](https://docs.openrewrite.org/) recipes for automated code transformations across YAML, Gradle, Java/Kotlin, and Spring projects. It wraps OpenRewrite's runtime into a CLI that can run recipes by name, integrate with GitHub Actions, and work with Dependabot PRs.

# Tech Stack

| Category | Technology |
|---|---|
| Primary language | Kotlin |
| Secondary language | Java (minimal, in `allwrite-recipes`) |
| Core framework | OpenRewrite |
| Dependency injection | Koin (with KSP annotation processing) |
| CLI framework | Clikt |
| Logging | SLF4J + Logback + kotlin-logging |
| Serialization | kotlinx-serialization |
| HTTP client | Ktor |
| Markdown rendering | Markout, clikt-markdown, Mordant |
| Build system | Gradle (Kotlin DSL) with composite build |
| Release management | JReleaser + Axion Release Plugin |
| Testing | Kotest (FunSpec), JUnit 5, MockK, OpenRewrite RewriteTest |

## Documentation

The documentation site is built with MkDocs from `mkdocs.yml`, `docs/requirements.txt`, and the `docs/` directory.

- `docs/index.md` — project overview
- `docs/cli.md` — CLI reference
- `docs/contributing.md` — recipe authoring guide
- `docs/recipes/index.md` — built-in recipes overview and authoring helpers
- `docs/recipes/{yaml,gradle,java,spring}/` — individual built-in recipe pages grouped by category
- `docs/extra.css` — documentation layout overrides

# Architecture

Multi-module Gradle project using **Hexagonal Architecture (Ports & Adapters)**.

## Module Dependency Graph

```
allwrite-cli  -->  allwrite-runtime  -->  allwrite-api
      |                  |                     ^
      |                  v                     |
      |            allwrite-recipes -----------+
      |                  |
      |                  v
      |            allwrite-spi
      |                  ^
      +-----> allwrite-completions
                    (annotation processor)
      |
      +-----> allwrite-recipes (direct, for recipe classpath)
```

## Module Responsibilities

| Module | Role |
|---|---|
| `allwrite-api` | Public API layer. Incoming port interfaces (`RecipeExecutor`, `RecipeSource`, `RecipeCoordinates`). Published as a Maven artifact. |
| `allwrite-spi` | Published SPI for recipe authors. Base classes (`AllwriteRecipe`, `AllwriteScanningRecipe`), `RecipeMetadata`, tag generation (including `dependabot-artifact`). |
| `allwrite-recipes` | Pure OpenRewrite recipe implementations. Published as a Maven artifact. Depends on `allwrite-api` and `allwrite-spi`. |
| `allwrite-runtime` | Domain layer. Outgoing port interfaces and OpenRewrite-backed implementations. Depends on `allwrite-api`. |
| `allwrite-cli` | Application + Infrastructure layer. CLI commands, OS/GitHub integration, DI wiring. |
| `allwrite-completions` | Build-time annotation processor for shell completion generation. |
| `build-logic` | Gradle composite build with convention plugins and custom tasks. |

# Directory Structure

```
allwrite/
├── allwrite-api/
│   └── src/main/kotlin/              Incoming port interfaces (RecipeExecutor, RecipeSource, RecipeCoordinates)
│
├── allwrite-cli/
│   ├── src/main/kotlin/              CLI application (commands, infrastructure adapters)
│   │   ├── runner/Main.kt            Entry point
│   │   ├── runner/RunnerModule.kt
│   │   ├── runner/application/       CLI commands (run, ls, external add/update/rm/ls), application logic
│   │   ├── runner/infrastructure/    OS, GitHub, PR manager adapters (incl. external recipe store)
│   │   └── runner/util/              Utility classes
│   ├── src/main/resources/           logback.xml
│   ├── src/test/kotlin/              Unit tests (Kotest FunSpec)
│   └── src/e2e/kotlin/              End-to-end tests
│
├── allwrite-runtime/
│   ├── src/main/kotlin/              Outgoing ports, recipe executor, source file parser
│   ├── src/test/kotlin/              Unit tests (Kotest FunSpec)
│   └── src/testFixtures/kotlin/      Shared test fakes/fixtures
│
├── allwrite-recipes/
│   ├── src/main/kotlin/
│   │   ├── recipes/yaml/            YAML transformation recipes
│   │   ├── recipes/spring/          Spring property/annotation and migration recipes, including conditional dependency and custom Spring Boot 4 migration helpers
│   │   ├── recipes/java/            Java refactoring recipes
│   │   ├── recipes/gradle/          Gradle dependency recipes
│   │   │   └── *DependencyRewriter.kt  Dedicated helpers for Gradle dependency transforms
│   │   ├── recipes/properties/      Properties file recipes
│   │   ├── recipes/toml/            TOML utilities
│   │   └── recipes/util/            Shared recipe utilities
│   ├── src/main/resources/META-INF/rewrite/   Declarative YAML recipes
│   ├── src/test/kotlin/              Unit tests (JUnit 5 + RewriteTest)
│   └── src/testFixtures/kotlin/      Test fixture classes
│
├── allwrite-spi/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [allegro/allwrite](https://github.com/allegro/allwrite) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
