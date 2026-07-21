---
trigger: always_on
description: This is a Java 21 + Scala 3 web application (WAR) for Linux.org.ru. It uses Maven for build,
---

# AGENTS.md - Development Guide for Linux.org.ru

## Project Overview

This is a Java 21 + Scala 3 web application (WAR) for Linux.org.ru. It uses Maven for build,
Spring Framework, and includes both unit and integration tests.

Project uses PostgreSQL 16 database, Opensearch 3.x for full text seach and analytics. It contains embedded
ActiveMQ.

## Build Commands

### Building the Project
```bash
mvn package -DskipTests    # Build WAR without running tests
mvn compile               # Compile main sources only
mvn test-compile          # Compile main and test sources
```

### Running Tests

**Run all unit tests only (no integration tests):**
```bash
mvn test
```

**Run a single test class:**
```bash
mvn test -Dtest=StringUtilTest
mvn test -Dtest=ru.org.linux.util.StringUtilTest
```

**Run a single test method:**
```bash
mvn test -Dtest=StringUtilTest#processTitle
```

**Run integration tests:**
```bash
mvn integration-test 
```

**Run a single integration test:**
```bash
mvn integration-test -Dit.test=TopicControllerIntegrationTest
```

**Run all tests (unit + integration):**
```bash
mvn verify
```

### Run application in development web server

Run in background shell:

```bash
mvn -DskipTests package jetty:run-war > server.log 2>&1 &
```

**Important:** The development web server must be restarted after any changes to the code.

Stop server with:

```bash
mvn jetty:stop
```

Server starts at http://127.0.0.1:8080/

All users in test development database has password 'passwd'. Use following users for testing:

* maxcom: administrator, full permissions
* svu: moderator
* edo: user (score >= 50)

### Test/Development Database

Integration tests and development web server uses pre-installed PostgreSQL at `postgresql:///lor`.

### JavaScript & CSS Build

This project uses **Maven-only** frontend tooling — no Node.js, npm, or package.json is required.

**CSS Pipeline:**
- Source: `src/main/webapp/sass/*.scss` (Sass source files)
- `dart-sass-maven-plugin` (phase: `generate-resources`) compiles Sass to CSS in compressed style
- `yuicompressor-maven-plugin` (phase: `generate-resources`) minifies CSS and aggregates per-theme `combined.css` bundles.
- `maven-war-plugin` excludes raw source CSS from the final WAR; only minified/aggregated copies are included

**JS Pipeline:**
- Custom JS source: `src/main/webapp/js/`
- `closure-compiler-maven-plugin` (phase: `generate-resources` / `process-resources`):
  - Merges `js/lor/*.js` into a single `lor.js`
  - Minifies individual files: `add-form.js`, `lor_view_diff_history.js`, `realtime.js`, `tagsAutocomplete.js`
  - Minifies individual plugins: `jquery.hotkeys.js`, `pattern.js`
- `maven-dependency-plugin` (phase: `generate-sources`) unpacks third-party JS libraries from WebJar dependencies
- `maven-antrun-plugin` (phase: `compile`) concatenates WebJar libraries into `plugins.js`
- Pre-minified files are copied as-is
- `maven-war-plugin` excludes raw source JS from the final WAR; only processed copies are included

### Other Commands
```bash
mvn clean                 # Clean target directory
mvn dependency:tree      # Show dependency tree
```

## Code Style Guidelines

### General

- All source files must include the Apache License header (see existing files)
- Maximum line length: 120 characters (enforced by Scalafmt)
- Package structure: `ru.org.linux.*`

### Java Conventions

- Use Spring annotations: `@Repository`, `@Service`, `@Controller`, etc.
- Use `@Nullable` and `@Nonnull` annotations from `javax.annotation`
- Use Java 17+ features (records, pattern matching) where appropriate
- Use `Optional` instead of null returns
- Use constructor injection over field injection
- Import order: java.*, javax.*, org.*, com.*, ru.*

### Scala Conventions

- Follow `.scalafmt.conf` configuration (version 3.10.7, Scala 3 dialect)
- Use strict logging: `com.typesafe.scalalogging.StrictLogging`
- Use Akka/Pekko for async operations
- Use `if then` / `if then else` instead of `if () {}` / `if () {} else {}`
- Use `match` with indentation-based syntax instead of curly braces where appropriate
- Use `end` markers for significant indentation blocks when clarity benefits
- Prefer `given`/`using` over `implicit`
- Use `extension` methods instead of implicit classes
- Prefer enums over sealed trait hierarchies for ADTs
- Use optional braces (significant indentation) consistently

### Naming Conventions

- Java classes: `CamelCase` (e.g., `UserDao`, `TopicController`)
- Scala classes/objects: `CamelCase` (e.g., `CommentCreateService`)
- Methods/fields: `camelCase`
- Java constants: `UPPER_SNAKE_CASE`
- Scala constants: `UpperCamelCase`
- Test classes: `*Test`, `*IntegrationTest`, `*WebTest.scala`

### Error Handling

- Use custom exceptions (e.g., `MessageNotFoundException`, `UserNotFoundException`)
- Return `Optional<T>` for potentially missing values
- Use Spring's `@ExceptionHandler` for controller-level error handling
- Log errors with appropriate levels (error for exceptions, info/debug for flow)

### Testing

- Use JUnit 4/5 for Java unit tests
- Use MUnit for Scala tests
- Follow AAA pattern (Arrange/Act/Assert or Given/When/Then)
- Place test classes in same package under `src/test/java` or `src/test/scala`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [maxcom/lorsource](https://github.com/maxcom/lorsource) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
