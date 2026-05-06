---
trigger: always_on
description: This is an IntelliJ IDEA plugin for Apache Kafka, providing a comprehensive UI for connecting to Kafka clusters,
---

# Confluent for JetBrains IDEs - AI Coding Agent Guide

## Project Overview

This is an IntelliJ IDEA plugin for Apache Kafka, providing a comprehensive UI for connecting to Kafka clusters,
producing/consuming messages, managing topics, and integrating with schema registries (Confluent and AWS Glue). Built
with the IntelliJ Platform Plugin SDK using Kotlin.

**Core Architecture**: Three-layer design (UI → KafkaDataManager → API Communication via KafkaClient)

## Essential Build & Development Commands

### Setup (Required First Step)

```bash
# Install SDKMAN and Java 24 (defined in .sdkmanrc)
sdk env install
```

### Development Workflow

```bash
# Build the plugin
./gradlew build

# Run plugin in development IDE instance
./gradlew runIde

# Run tests (JUnit 5)
./gradlew test

# Build distributable ZIP
./gradlew buildPlugin  # Output: build/distributions/
```

Run `./gradlew clean` to remove dependencies from prior builds before running `./gradlew build`, when you've updated the build tooling, after larger refactors, or when old changes keep lingering around longer than expected.

### Secrets for Telemetry (Sentry/Segment)

**For production builds only** - local development does not require telemetry setup.

Required environment variables: `SENTRY_AUTH_TOKEN`, `SENTRY_DSN`, `SEGMENT_WRITE_KEY`

```bash
# From Vault (Confluent internal - only needed for production builds)
vault_login
. scripts/get-secrets.sh
```

Without these environment variables, telemetry tasks are automatically disabled but builds should still work fine for
local development.

### CI/Build Cache

Uses Confluent's `cc-mk-include` system. Makefile auto-downloads required mk files from GitHub.

## Architecture Patterns

### Plugin Entry Points

1. **Tool Window**: `KafkaToolWindowFactory` → `KafkaMonitoringToolWindowController` → `KafkaMainController`
2. **Editors**: `KafkaEditorProvider` creates `KafkaProducerEditor` and `KafkaConsumerEditor`
3. **Actions**: Defined in `plugin.xml` under `<actions>` groups (`Kafka.Topic.Actions`, `Kafka.Schema.Actions`, etc.)

### Controller Pattern

Controllers manage UI components and coordinate with `KafkaDataManager` (which extends `MonitoringDataManager`):

- `KafkaMainController` - Main tree view and navigation
- `TopicsController`, `TopicDetailsController` - Topic management
- `ConsumerGroupsController`, `ConsumerGroupOffsetsController` - Consumer groups
- `KafkaRegistryController`, `KafkaSchemaController` - Schema registry
- `ConfluentMainController`, `ConfluentTabController` - Confluent Cloud

All controllers extend base types and follow `Disposable` pattern (register with `Disposer.register(parent, child)`).

### Data Context Mechanism

Actions access data via `DataKey` extension properties:

```kotlin
// In controller companion object
val DATA_MANAGER: DataKey<MonitoringDataManager> = DataKey.create("kafka.data.manager")

// Extension property for easy access
val AnActionEvent.dataManager
get() = dataContext.getData(DATA_MANAGER)
```

### Data Models

Domain objects use `*Presentable` suffix (e.g., `TopicPresentable`, `ConsumerGroupPresentable`):

- Located in `io.confluent.intellijplugin.model`
- Annotated with rendering hints: `@NoRendering`, `@LoadingRendering`
- Include companion object with `renderableColumns` for table display
- Define localization keys referencing `KafkaBundle.properties`

### Localization

All user-facing strings use `KafkaMessagesBundle.message("key")` with keys in
`resources/messages/KafkaBundle.properties`.
Use `@Nls` annotation for localized strings, `@NlsSafe` for non-localized (e.g., technical identifiers).

## Code Generation & Build-Time Tasks

Build generates configuration files from environment variables:

- `SentryConfig.kt` - Embeds `SENTRY_DSN` at compile time
- `SegmentConfig.kt` - Embeds `SEGMENT_WRITE_KEY` at compile time

Generated sources go to `build/generated/sources/{sentryconfig,segmentconfig}/kotlin` and are included in
`compileKotlin`.

## Testing Conventions

- **Framework**: JUnit 5 (with JUnit 4 runtime for platform compatibility)
- **Test Location**: `test/io/confluent/intellijplugin/`
- **Platform Tests**: Use `@TestApplication` annotation for IntelliJ platform context
- **Mocking**: `mockito-kotlin` for mock objects

Example test structure:

```kotlin
@TestApplication
class MyFeatureTest {
    @Test
    fun `descriptive test name in backticks`() {
        // Arrange, Act, Assert
    }
}
```

## Spring Boot Integration

Optional dependency (declared in `plugin.xml`):

```xml

<depends config-file="spring-boot.xml" optional="true">com.intellij.spring.boot</depends>
```

When Spring Boot plugin is present, adds:

- Gutter icons in `application.properties`/`application.yml` for Kafka configuration
- Ability to create connections from Spring config files
- Line markers via `KafkaSpringBootConfigLineMarkers`

## Configuration & Settings

- **Plugin settings**: `plugin.xml` (declarations, extensions, actions)
- **Gradle config**: Version catalog in `gradle/libs.versions.toml`
- **JVM target**: Java 21 (defined in `build.gradle.kts`)
- **Kotlin compiler args**: `-Xjvm-default=all`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [confluentinc/intellij](https://github.com/confluentinc/intellij) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
