---
trigger: always_on
description: Generates API interfaces, controllers, and service interfaces.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Zygarde is a multi-module Kotlin framework for simplifying enterprise application development. It provides code generation (KAPT and DSL-based), JPA enhancements with type-safe search DSL, web/REST utilities, and model mapping capabilities for building Kotlin/Spring applications.

**Tech Stack**: Kotlin 1.8.22, Spring Boot 2.7.14, Gradle 7.x+ (Kotlin DSL), JUnit Platform, Kotest, MockK

## Essential Commands

### Build & Test
```bash
# Full build with tests and coverage
./gradlew build

# Run tests only
./gradlew test

# Test specific module
./gradlew -p modules-core/zygarde-core test

# Run sample application
./gradlew :samples:todo-legacy:bootRun
```

### Code Quality
```bash
# Auto-format code (do this before committing)
./gradlew ktlintFormat

# Check formatting
./gradlew ktlintCheck

# Run static analysis
./gradlew detekt

# Coverage is automatically reported after test runs
```

### Development
```bash
# Refresh dependency locks
./scripts/deps.sh

# Run single test class
./gradlew test --tests "ClassName"

# Run KAPT code generation
./gradlew kaptKotlin
```

## Module Architecture

### Module Structure
```
zygarde/
├── modules-core/           # Core utilities (error-handling, DI, Jackson, JWT, mail, etc.)
├── modules-jpa/            # JPA extensions, DAOs, search DSL, Envers support
├── modules-web/            # Web/REST utilities (WebMVC, WebFlux, security)
├── modules-model-mapping/  # Object mapping DSL and code generation
├── modules-codegen-support/# Code generation infrastructure and base classes
├── modules-test-support/   # Shared test fixtures and utilities
├── modules-bom/            # Bill of Materials for dependency management
├── samples/                # Example applications and generated code validation
└── doc/                    # Design documentation
```

**Convention**: Each module follows standard Kotlin layout:
- `src/main/kotlin/` - production code
- `src/test/kotlin/` - unit tests
- `build.gradle.kts` - module build config

**Pattern**: Codegen modules (`*-codegen`) contain annotation processors; runtime modules contain base classes/interfaces/implementations.

### Automatic Module Registration
Modules are auto-discovered in `settings.gradle.kts` via `registerModules()` function. Just add a directory with `build.gradle.kts` under `modules-*/` and it's included.

## Code Generation Architecture

### Two-Tier Code Generation System

#### 1. KAPT-Based (Annotation Processors)
Compile-time annotation processing for JPA entities:

**Key Annotation**: `@ZyModel` (place on `@Entity` classes)

**Generated Artifacts**:
- Entity field search extensions (DSL helper functions like `name()`, `author()`)
- DAO interfaces with configurable inheritance
- Combined `Dao` component aggregating all DAOs

**KAPT Configuration** (in `build.gradle.kts`):
```kotlin
kapt {
  arguments {
    arg("zygarde.codegen.base.package", "my.package.codegen")
    arg("zygarde.codegen.dao.inherit", "zygarde.data.jpa.dao.ZygardeEnhancedDao")
  }
}
```

**Key Options**:
- `zygarde.codegen.base.package` - base package for generated code
- `zygarde.codegen.dao.inherit` - custom base DAO class
- `zygarde.codegen.dao.suffix` - DAO naming suffix (default: "Dao")

#### 2. DSL-Based (Standalone Codegen)
Runtime DSL scripts that generate complete application layers:

**Model Mapping DSL** (`ModelMappingCodegenSpec`):
```kotlin
class MyModelDsl : ModelMappingCodegenSpec({
  MyDto {
    fromAutoIntId(Entity::id)      // Entity→DTO mapping
    from(Entity::description)
  }

  CreateMyReq {
    applyTo(Entity::description)   // DTO→Entity mapping
  }
})
```

**Web API DSL** (`WebMvcDslCodegen`):
Generates API interfaces, controllers, and service interfaces.

**Code Generation Workflow**:
1. Modify DSL definitions or annotation processors
2. Run code generation (KAPT or DSL script)
3. Regenerate sample outputs in `samples/todo-multimodule-dsl`
4. Review generated code diffs in version control
5. Ensure generated code compiles and passes tests

**IMPORTANT**: Never manually edit generated files. Keep them isolated in dedicated `*-codegen` modules or `doc/codegen-*` directories.

## JPA Extensions & Search DSL

### Type-Safe Search DSL Pattern

Zygarde provides a type-safe, fluent DSL for JPA queries that abstracts JPA Criteria API complexity:

```kotlin
// Generated DAO usage
bookDao.search {
  name() eq "MyBook"
  author().age() gt 30
  status() inList listOf(Status.ACTIVE, Status.PENDING)
}

// Enhanced DAO operations (if using ZygardeEnhancedDao)
bookDao.remove {
  name() eq "MyBook"
}
```

**Architecture**:
- `BaseDao<T, ID>` - combines `JpaRepository` + `JpaSpecificationExecutor`
- `EnhancedSearch<T>` - interface defining DSL operations
- `EnhancedSearchImpl<T>` - JPA Criteria implementation with predicate building
- Generated extension functions (e.g., `name()`, `author()`) return typed action objects

**Action Types**:
- `ConditionAction<Root, Current, Field>` - generic operations (eq, notEq, inList, isNull)
- `StringConditionAction` - string operations (contains, startsWith, like)
- `ComparableConditionAction` - numeric/comparable operations (gt, lt, gte, lte)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/zygarde-projects) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
