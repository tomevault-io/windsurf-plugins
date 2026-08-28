---
trigger: always_on
description: Java code generation standards and testing guidelines for hexagonal architecture - Synanton platform
---


# Java Generation Rules

## Role

You are a senior developer, obsessed with clean and fully tested code.

## Code Structure

Use hexagonal architecture packages pattern. The project structure follows this pattern:

```
src/
├── adapter/
│   ├── in/           # Entry points (incoming adapters)
│   │   ├── grpc/     # gRPC controllers/endpoints
│   │   ├── rest/     # REST controllers
│   │   ├── schedule/ # @Scheduled (and Shedlock) triggers - delegate to domain use cases only
│   │   └── ...       # Other entry point adapters (e.g. kafka, messaging)
│   └── out/          # Outgoing adapters (external resources)
│       ├── database/ # Database repositories implementations
│       ├── client/   # External service clients
│       └── ...       # Other outgoing adapters
├── domain/           # Core business logic
│   ├── service/      # Domain services
│   ├── model/        # Domain models/entities
│   └── [UseCase classes directly here, e.g., GetAllTemplatesUseCase.java]
└── config/           # Configuration classes and files
```

**Adapter package:**
- **`adapter/in/`**: Contains all entry points for the application. These are adapters that receive external requests (gRPC endpoints, REST controllers, Kafka listeners, scheduled jobs under `schedule/` that only invoke use cases).
- **`adapter/out/`**: Contains all outgoing adapters that interact with external resources (database repository implementations, external API clients, Kafka publishers).

**Domain package:**
- Contains all business logic, services, use cases (directly in domain package, not in subfolder), and domain models. This is the core of the application and must not depend on adapters.

**Config package:**
- Contains Spring configuration classes, bean definitions, and other configuration files.

## Build System

**Synanton uses Gradle with Kotlin DSL** (`build.gradle.kts`). The wrapper is `./gradlew`.

Key build commands:
```bash
./gradlew compileJava          # compile before making changes
./gradlew test                 # run all tests
./gradlew :java:<module>:test  # run tests for a specific module
./gradlew :java:<module>:bootRun  # start a specific service
./gradlew build                # full build (compile + test + package)
```

**Module layout** (multi-module Gradle build):
- `java/<module>/build.gradle.kts` - module build file
- `settings.gradle.kts` - lists all modules
- `build.gradle.kts` - root build file with shared config

Use `./gradlew dependencies --configuration compileClasspath` to inspect the dependency tree for a module.

## Guidelines

When generating Java code:
- Make minimum change possible to complete the task
- Create multi-char variable names (no single-char variable names)
- When generating POJOs, use Lombok annotations to avoid boilerplate getters/setters/constructors
- Prefer `@RequiredArgsConstructor` (Lombok) for dependency injection and `final` field initialization whenever it fits; avoid hand-written constructors that only assign fields
- Use `rows.getFirst()` instead of `rows.get(0)` where possible
- Use `@Accessors(chain = true)` from `lombok.experimental.Accessors` on domain classes for fluent construction
- Repository classes should return domain objects, not protobuf objects
- Use case classes should return domain objects; protobuf conversion happens in the gRPC adapter layer
- Protobuf class fields cannot be null - no need for null checks on them
- Annotate every field and method parameter that may be `null` with `@Nullable` (`import org.jspecify.annotations.Nullable`); unannotated reference types are treated as non-null. Skip protobuf fields and test code.

**Configuration defaults (Spring Boot):**
- Do not put tunable defaults in Java: no `@Value("${key:DEFAULT}")`, no literal config fallbacks.
- Put defaults in `application.yml` (including per-profile files) and in test `application.yml` when tests require the property.
- Inject resolved values only: `@Value("${key}")` or `@ConfigurationProperties` backed by YAML.

**Code Style:**
- Respect checkstyle rules in `/checkstyle.xml`
- Maximum line length: 120 characters (as configured in checkstyle.xml)
- Prefer multi-line builder pattern over one-liners for readability

```java
// Instead of this:
AssistantsOutProto.Error.newBuilder().setCode(failure.message()).setMessage(failure.message()).build();

// Do this:
AssistantsOutProto.Error.newBuilder()
    .setCode(failure.message())
    .setMessage(failure.message())
    .build();
```

**Comments and Javadoc:**
- Do not edit or remove existing comments/Javadoc unless they are wrong or contradict the new code
- Add Javadoc on public classes, methods, and fields that form the module API
- Add brief comments only for non-obvious logic (invariants, external system contracts, multi-step algorithms)
- Do not restate the code; do not document volatile implementation details unless required for safety or a stable contract
- Prefer stable "why/what" wording over step-by-step narration

**Logging:**
- Use `@Slf4j` annotation with log methods instead of `System.out.print`
- No need to check logging level before logging

```java
// Instead of this:
if (log.isDebugEnabled()) {
    log.debug("IDs to process: {}", ids);
}

// Do this:
log.debug("IDs to process: {}", ids);

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [synanton/platform](https://github.com/synanton/platform) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
