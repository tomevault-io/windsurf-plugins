---
trigger: always_on
description: This document contains important information for Claude AI assistant when working on this project.
---

# Claude Development Documentation

This document contains important information for Claude AI assistant when working on this project.

## Project Overview

This is a JUnit 5 workshop project demonstrating modern Java testing practices. It's a multi-project Gradle build showcasing JUnit 5 features, Spring Boot integration, and JUnit 4 legacy support for training purposes.

## Build System & Dependencies

### Core Technologies
- **Language**: Java 17+ (uses Gradle toolchain)
- **Build Tool**: Gradle 8.14.2 (latest stable)
- **Testing**: JUnit 5.13.2 (with BOM management)
- **CI**: GitHub Actions

### Key Dependencies
- `org.junit.jupiter:junit-jupiter` - Main testing framework (via BOM)
- `org.junit.platform:junit-platform-launcher` - Required for test execution
- `org.mockito:mockito-core` & `org.mockito:mockito-junit-jupiter` - Mocking framework
- `org.assertj:assertj-core` - Fluent assertions (3.27.3)
- `net.jqwik:jqwik` - Property-based testing (1.9.3)

### Dependency Management
- Uses Gradle version catalogs (`gradle/libs.versions.toml`)
- JUnit BOM manages all JUnit component versions consistently
- Main project uses JUnit 5 only (no vintage engine)
- Vintage subproject specifically uses JUnit 4 via vintage engine

## Project Structure

```
junit5_workshop/
├── src/                    # Main project - JUnit 5 examples
│   ├── main/java/          # Demo classes and utilities
│   └── test/java/          # JUnit 5 test suite (30+ test files)
├── spring/                 # Spring Boot integration subproject
│   ├── src/main/           # Spring Boot application with JPA
│   └── src/test/           # Spring-specific tests
├── vintage/                # JUnit 4 legacy demonstration
│   └── src/test/           # JUnit 4 tests using vintage engine
└── gradle/
    └── libs.versions.toml  # Version catalog for dependencies
```

## Multi-Project Configuration

### Main Project (`/`)
- **Purpose**: Primary JUnit 5 feature demonstrations
- **Dependencies**: Uses JUnit BOM approach, no vintage engine
- **Test Configuration**: Excludes 'slow' tests, parallel execution enabled

### Spring Subproject (`/spring`)
- **Purpose**: Spring Boot + JUnit 5 integration examples
- **Framework**: Spring Boot 3.5.3 with Spring Test
- **Database**: H2 in-memory database with JPA entities
- **Testing**: Repository tests, database assertions with AssertJ-DB

### Vintage Subproject (`/vintage`)
- **Purpose**: JUnit 4 legacy support demonstration for training
- **Configuration**: Uses vintage engine only, excludes Jupiter engine
- **Important**: Keep vintage engine for educational purposes

## Recent Modernization Work

### Major Updates Completed (June 2025)
1. **Gradle Upgrade**: 8.11.1 → 8.14.2
2. **JUnit BOM Implementation**: Switched main project to BOM approach
3. **Dependency Updates**: All testing libraries updated to latest stable versions
4. **GitHub Actions**: Updated to latest action versions (@v4)

### Testing Framework Architecture
- **Main project**: Pure JUnit 5 with BOM management
- **Spring project**: Spring Boot Test + JUnit 5
- **Vintage project**: JUnit 4 via vintage engine (intentionally preserved)

## Common Development Tasks

### Running Tests
```bash
# All projects
./gradlew test

# Specific projects
./gradlew :spring:test
./gradlew :vintage:test

# With detailed output
./gradlew test --info
```

### Building
```bash
# Clean build all projects
./gradlew clean build

# Just compile
./gradlew compileJava compileTestJava
```

### Code Formatting
```bash
./gradlew spotlessApply
```

## Important Notes for Claude

### Multi-Project Considerations
- **NEVER remove vintage engine** from shared catalog - it's needed for vintage subproject
- Main project should NOT use vintage engine (removed from its dependencies)
- Spring subproject uses Spring Boot's managed dependencies
- Each subproject has its own build.gradle with specific test configurations

### When Working with Tests
- Main project: All tests use JUnit 5 - no JUnit 4 imports
- Spring project: Use Spring Boot Test annotations and slices
- Vintage project: Deliberately uses JUnit 4 patterns for training
- Use `assertThrows()` for exception testing in JUnit 5 tests
- Use `@BeforeEach/@AfterEach` instead of `@Before/@After`

### Code Patterns to Follow
- Java 17+ features available but maintain educational clarity
- Use BOM approach for dependency management in main project
- Follow existing package structure and naming conventions
- Preserve different testing styles in each subproject for educational value

### Dependencies
- Use version catalog (`gradle/libs.versions.toml`) for version management
- JUnit BOM ensures version consistency across JUnit components
- Spring Boot manages its own dependency versions
- Never modify vintage engine setup (it's intentional for training)

### GitHub Actions
- CI runs on Java 17
- Builds with `./gradlew build` (all subprojects)
- All tests must pass for successful builds

## Known Project Characteristics

### Test Organization
- Main project: Comprehensive JUnit 5 feature coverage
- Parallel test execution enabled for performance
- 'Slow' tests tagged and excluded by default
- Multiple testing libraries demonstrated (AssertJ, Mockito, jqwik)

### Educational Structure

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kousen/junit5_workshop](https://github.com/kousen/junit5_workshop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
