---
trigger: always_on
description: IntelliJ plugin that provides IDE support for Axon Framework 4.x and 5.x. Automatically detects project version and adapts behavior. Provides handler detection, navigation, and code validation for both Java and Kotlin.
---

# Axon Framework IntelliJ Plugin - Technical Overview

## Quick Summary

IntelliJ plugin that provides IDE support for Axon Framework 4.x and 5.x. Automatically detects project version and adapts behavior. Provides handler detection, navigation, and code validation for both Java and Kotlin.

**Version:** 0.9.5
**Test Suite:** 159 tests (140 v4, 19 v5) - all passing
**Min IntelliJ:** 2024.2

---

## Key Architecture Concepts

### Version Detection & Adaptation

**AxonVersionService** (`/usage/AxonVersionService.kt`):
- Scans JAR dependencies on project open
- Reads `META-INF/maven/*/pom.properties` to extract version
- Creates appropriate `VersionedComponentFactory` (v4 or v5)
- Re-checks on dependency changes only

**Factory Pattern:**
```
VersionedComponentFactory
├── Axon4ComponentFactory → Returns v4 searchers (Saga, Deadline, AggregateConstructor)
└── Axon5ComponentFactory → Returns v5 searchers (EntityCreator only)
```

This eliminates need for version checks scattered throughout code.

### Handler Discovery

**Flow:**
1. `AnnotationResolver` scans for Axon annotations (both v4 & v5 packages, meta-annotations up to 5 levels)
2. `MessageHandlerResolver` gets version-appropriate searchers from factory
3. Searchers (CommandHandler, EventHandler, etc.) scan PSI for annotated methods
4. Results cached using PSI modification tracking

**Handler Types:**
- **v4 & v5:** CommandHandler, EventHandler, EventSourcingHandler, QueryHandler
- **v4 only:** SagaEventHandler, DeadlineHandler, AggregateConstructor (implicit)
- **v5 only:** EntityCreator (explicit)

### Key Differences: v4 vs v5

**Package Changes (v5):**
- Handlers moved to `org.axonframework.messaging.*.annotation` packages
- EventSourcingHandler: `org.axonframework.eventsourcing.annotation`

**Annotation Changes (v5):**
```java
// Aggregates
@AggregateRoot → @EventSourcedEntity (or @EventSourced for Spring)
@AggregateMember → @EntityMember

// Handlers
Constructor → @EntityCreator (must be explicit)

// Removed
@Saga, @SagaEventHandler, @DeadlineHandler (features removed in v5)
```

**Critical:** Spring users use `@EventSourced` stereotype which is meta-annotated with `@EventSourcedEntity`.

---

## Project Structure

```
/src/main/kotlin/.../plugin/
├── api/                    # AxonAnnotation, MessageHandlerType, Handler types
├── usage/                  # AxonVersionService + component factories
├── resolving/
│   ├── AnnotationResolver.kt       # Scans annotations (v4 & v5)
│   ├── MessageHandlerResolver.kt   # Gets handlers from factory
│   ├── AggregateStructureResolver.kt  # Entity hierarchy
│   └── handlers/searchers/         # 8 searcher implementations
├── markers/                # Line marker providers (gutter icons)
└── inspections/            # Code checks (v4 only - disabled for v5)
```

---

## Important Implementation Details

### AxonAnnotation Enum (`/api/AxonAnnotation.kt`)

Defines all annotations with both v4 and v5 fully qualified names:
```kotlin
COMMAND_HANDLER(
    "org.axonframework.commandhandling.CommandHandler",  // v4
    "org.axonframework.messaging.commandhandling.annotation.CommandHandler"  // v5
),
EVENT_SOURCED(
    null,  // No v4 equivalent
    "org.axonframework.extension.spring.stereotype.EventSourced"  // v5 Spring
),
AGGREGATE_MEMBER(
    "org.axonframework.modelling.command.AggregateMember",  // v4
    "org.axonframework.modelling.entity.annotation.EntityMember"  // v5
),
```

**Note:** `EVENT_SOURCED` points to Spring stereotype, `EVENT_SOURCED_ENTITY` points to core annotation. Both resolve to same behavior via meta-annotation scanning.

### Aggregate Recognition (`/util/PSiProcessingUtils.kt`)

```kotlin
fun PsiClass?.isAggregate(): Boolean {
    if (this == null) return false
    return isAnnotated(AGGREGATE_ROOT) ||      // v4
           isAnnotated(EVENT_SOURCED) ||        // v5 Spring
           isAnnotated(EVENT_SOURCED_ENTITY)    // v5 Core
}
```

### Line Marker Caret Position

Line markers detect annotations by looking for `UAnnotation` parent. Caret must be **inside** annotation:
```kotlin
@CommandHandler<caret>  // ✅ Works
fun handle() {}

@CommandHandler
fun <caret>handle() {}  // ❌ Doesn't work
```

This is why line marker tests place caret inside annotations.

---

## Test Infrastructure

### Dual Test Base Classes

**Why needed:** Gradle resolves to single version, but we need both v4 and v5 for tests.

**Solution:** Download JARs from Maven Central, cache locally, add via `PsiTestUtil`.

**Version Configuration:** Versions configured in `gradle.properties`:
```properties
axonVersion=4.10.1    # v4 test version
axon5Version=5.0.0    # v5 test version
```

Passed to tests via system properties (see `build.gradle.kts`):
```kotlin
test {
    systemProperty("axonVersion", properties("axonVersion"))
    systemProperty("axon5Version", properties("axon5Version"))
}
```

**AbstractAxonFixtureTestCase** (v4 tests):
- Reads `System.getProperty("axonVersion")` (defaults to "4.10.1")
- Downloads JARs to `src/test/resources/libraries/`
- Adds via `PsiTestUtil.addLibrary()` (test scope)
- 140 tests

**AbstractAxon5FixtureTestCase** (v5 tests):

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AxonFramework/IdeaPlugin](https://github.com/AxonFramework/IdeaPlugin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
