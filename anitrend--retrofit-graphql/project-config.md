---
trigger: always_on
description: <!-- code-review-graph MCP tools -->
---

<!-- code-review-graph MCP tools -->
## MCP Tools: code-review-graph

**IMPORTANT: This project has a knowledge graph. ALWAYS use the
code-review-graph MCP tools BEFORE using Grep/Glob/Read to explore
the codebase.** The graph is faster, cheaper (fewer tokens), and gives
you structural context (callers, dependents, test coverage) that file
scanning cannot.

### When to use graph tools FIRST

- **Exploring code**: `semantic_search_nodes` or `query_graph` instead of Grep
- **Understanding impact**: `get_impact_radius` instead of manually tracing imports
- **Code review**: `detect_changes` + `get_review_context` instead of reading entire files
- **Finding relationships**: `query_graph` with callers_of/callees_of/imports_of/tests_for
- **Architecture questions**: `get_architecture_overview` + `list_communities`

Fall back to Grep/Glob/Read **only** when the graph doesn't cover what you need.

### Key Tools

| Tool | Use when |
|------|----------|
| `detect_changes` | Reviewing code changes — gives risk-scored analysis |
| `get_review_context` | Need source snippets for review — token-efficient |
| `get_impact_radius` | Understanding blast radius of a change |
| `get_affected_flows` | Finding which execution paths are impacted |
| `query_graph` | Tracing callers, callees, imports, tests, dependencies |
| `semantic_search_nodes` | Finding functions/classes by name or keyword |
| `get_architecture_overview` | Understanding high-level codebase structure |
| `refactor_tool` | Planning renames, finding dead code |

### Workflow

1. The graph auto-updates on file changes (via hooks).
2. Use `detect_changes` for code review.
3. Use `get_affected_flows` to understand impact.
4. Use `query_graph` pattern="tests_for" to check coverage.

---

# Retrofit GraphQL — Project Context

## Overview

**Retrofit GraphQL** is a converter library for Retrofit that enables injection of `.graphql` query/mutation files into HTTP request bodies with GraphQL variables. It bridges Retrofit's HTTP capabilities with GraphQL's query-based approach, letting Android developers work with raw `.graphql` files while keeping full control over their model classes.

### Architecture Principles

- **Converter Pattern** — Core functionality is a Retrofit `Converter.Factory` that transforms annotated method calls into GraphQL HTTP requests.
- **File-Based + Optional Code Generation** — Supports both runtime `.graphql` file loading from assets (via `@GraphQuery` annotation) AND build-time code generation via a Gradle plugin. Developers choose between flexible hand-written models and type-safe generated helpers.

## Module Organization

The project is organized into composable modules under the `co.anitrend.retrofit.graphql` package root:

| Module | Type | Purpose |
|--------|------|---------|
| `:annotations` | Kotlin JVM | `@GraphQuery` annotation (runtime retention) |
| `:api` | Android | Public API interfaces: `GraphQLOperation`, `GraphQLDocumentRegistry`, `QueryContainerBuilder`, `GraphQLRequest`, `GraphQLVariables`, `GraphContainer` |
| `:android-assets` | Android | Runtime asset-based query discovery: `GraphProcessor`, `AssetManagerDiscoveryPlugin`, APQ, logging |
| `:runtime` | Android | Retrofit `Converter.Factory`: `GraphConverter`, `GraphRequestConverter`, `GraphResponseConverter`. Depends on `:api`, `:android-assets`, `:annotations` |
| `:codegen-core` | Kotlin JVM | Code generation engine: parses `.graphql` files with graphql-java, generates Kotlin with KotlinPoet. Uses `SchemaIndex` for typed schema metadata, `GraphQLTypeMapper` for kind-aware mapping, `GraphQLDefaultValueRenderer` for default literals, and `GraphQLTypeUsageValidator` for recursive scalar validation. |
| `:gradle-plugin` | Gradle Plugin | Plugin `id("co.anitrend.retrofit.graphql.codegen")`. Registers `GenerateGraphQLSourcesTask`, wires output into source sets, owns standalone functional tests, and publishes its own marker/implementation artifacts from the composite build. |
| `:serialization-gson` | Android | Gson-backed `GraphQLJson` serialization |
| `:serialization-kotlinx` | Android | kotlinx.serialization-backed `GraphQLJson` |
| `:library` | Android | **Deprecated** aggregator. Re-exports all modules via `api()`. Contains type aliases from old `io.github.wax911.library` package |
| `:app` | Android | Sample GitHub API client demonstrating both asset-based and codegen workflows. Uses generated registry, typed request helpers, and explicit multipart upload. |

### Dependency Graph

```
:app → :runtime → :api + :android-assets + :annotations (android-assets and annotations pulled transitively via :runtime)
:app → :serialization-gson/:serialization-kotlinx (optional)
:app → :gradle-plugin → :codegen-core (build-time only, generates typed operation helpers)
:library (deprecated) → api() aggregates all modules above
```

## Build Logic & Conventions

- **Shared Gradle plugin**: `co.anitrend.retrofit.graphql` (`buildSrc`) applies Android config, Dokka, Spotless, publishing, and dependency strategies. Avoid duplicating configuration in individual modules.
- **Version catalog**: `gradle/libs.versions.toml`. Add new deps there first, then reference by alias.
- **Toolchain**: Java/Kotlin 21 (pinned in `.java-version`). Local dev uses `jenv`; CI uses `actions/setup-java`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AniTrend/retrofit-graphql](https://github.com/AniTrend/retrofit-graphql) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
