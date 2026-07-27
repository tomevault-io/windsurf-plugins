---
trigger: always_on
description: This document provides essential information for AI agents working on the iCal4j codebase.
---

# AGENTS.md - iCal4j Development Guide

This document provides essential information for AI agents working on the iCal4j codebase.

## Project Overview

iCal4j is a Java library for reading and writing iCalendar (RFC5545) data streams. It provides:
- Parser and builder for .ics files
- Object model for calendar components (events, todos, journals, etc.)
- Timezone support with Olson database integration
- Validation and transformation utilities
- Groovy DSL for calendar building

**Key Stats:**
- ~572 Java files + 148 Groovy files
- Java 11+ target (currently supports Java 17, 21)
- Uses Gradle build system
- Comprehensive test suite with Spock Framework

## Essential Commands

### Build and Test
```bash
# Build the project
./gradlew build

# Run tests only
./gradlew test

# Run all checks (includes tests, validation, style checks)
./gradlew check

# Clean build
./gradlew clean

# Install to local Maven repository
./gradlew publishToMavenLocal
```

### Using Makefile (Wrapper)
```bash
# Recommended approach - uses Makefile wrapper
make clean          # Clean build
make test          # Run tests
make build         # Build project
make check         # Run all checks
make install       # Install to local repo
make verify        # Verify build
```

### Code Quality
```bash
# API compatibility checks
./gradlew revapi
make listApiChanges

# Coverage reports (enabled by default)
# Results in: build/reports/jacoco/test/html/index.html
```

### Versioning and Release
```bash
# Check current version
./gradlew -q currentVersion
make currentVersion

# Mark next version (for maintainers - optional as will auto-increment)
make markNextVersion <version>

# Tag and push release
make release

# Publish release to Maven Central
make publish
```

## Project Structure

```
src/
├── main/java/net/fortuna/ical4j/
│   ├── model/              # Core object model
│   │   ├── component/      # Calendar components (VEvent, VTodo, etc.)
│   │   ├── property/       # Calendar properties (DtStart, Summary, etc.)
│   │   └── parameter/      # Property parameters (TzId, Value, etc.)
│   ├── data/              # Parsing and output
│   ├── validate/          # Validation framework
│   ├── filter/            # Filtering and querying
│   ├── transform/         # Data transformation
│   ├── util/              # Utilities
│   └── agent/             # User agent implementations
├── main/groovy/           # Groovy DSL and extensions
├── test/java/             # Java tests
├── test/groovy/           # Groovy/Spock tests
└── test/resources/        # Test resources and sample files
```

## Code Patterns and Conventions

### Naming Conventions
- **Classes**: PascalCase (`VEvent`, `DtStart`, `CalendarBuilder`)
- **Properties**: camelCase (`dtStart`, `lastModified`)
- **Constants**: UPPER_SNAKE_CASE (`PROPERTY_NAME`)
- **Packages**: lowercase dot notation (`net.fortuna.ical4j.model`)

### Property and Component Structure
- Properties extend `Property` base class
- Components extend `Component` base class
- Immutable variants exist for core properties (in `immutable/` packages)
- Factory pattern used extensively for object creation

### Code Organization
- Model classes follow RFC5545 structure closely
- Property names match iCalendar specification (DTSTART → DtStart)
- Parameter names often use underscore suffix in DSL (`tzid_` for TZID parameter)

### License Headers
All Java files include BSD-3-Clause license header:
```java
/**
 * Copyright (c) 2012, Ben Fortuna
 * All rights reserved.
 * ...
 */
```

### Documentation
- Extensive Javadoc with RFC references
- Properties include RFC section quotes in class documentation
- Package-level documentation in `package-info.java` files

## Testing Framework

### Test Structure
- **Java Tests**: JUnit 5 for unit tests
- **Groovy Tests**: Spock Framework for BDD-style tests
- **Test Resources**: Sample .ics files in `src/test/resources/`

### Test Naming
- Groovy specs use `SomeClassSpec.groovy` pattern
- Java tests use `SomeClassTest.java` pattern
- Test methods descriptive: `"test Component.calculateRecurrenceSet"`

### Common Test Patterns
```groovy
// Spock data-driven tests
where:
period | expectedResults
value1 | result1
value2 | result2

// Content builder for test data
ContentBuilder builder = []
VEvent event = builder.vevent {
    dtstart '20240101T140000'
    summary 'Test Event'
}
```

## Gradle Configuration

### Key Build Features
- **Java Features**: Multiple optional feature sets (groovyDsl, schemaValidation, etc.)
- **Testing**: JUnit Platform with Spock integration
- **Coverage**: Jacoco with 70% minimum threshold (non-failing)
- **Publishing**: Maven Central deployment
- **Signing**: GPG signing for releases

### Dependencies
- **Core**: SLF4J, Commons Lang3, Commons Collections4, ThreeTen-Extra
- **Optional**: Caffeine (caching), JParsec (expressions), Groovy (DSL)
- **Test**: Spock, JUnit 5, Hamcrest, Testcontainers

### Configuration Files
- `gradle.properties` - Build configuration
- `gradle/libs.versions.toml` - Version catalog
- `settings.gradle` - Project settings

## Development Workflows

### Making Changes
1. **Read existing code** - Follow established patterns
2. **Check RFC compliance** - Properties/components must match specifications

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ical4j/ical4j](https://github.com/ical4j/ical4j) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
