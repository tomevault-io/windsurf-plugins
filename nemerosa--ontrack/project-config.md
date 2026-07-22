---
trigger: always_on
description: Yontrack (formerly Ontrack) is a **continuous delivery monitoring platform**. It tracks projects,
---

# Yontrack (Ontrack) Developer Guide for Claude Code

Yontrack (formerly Ontrack) is a **continuous delivery monitoring platform**. It tracks projects,
branches, builds, promotions, and validations across the software delivery lifecycle.

## Tech Stack

- **Backend:** Kotlin + Java, Spring Boot 3.5+, Spring GraphQL, Spring Security
- **Frontend:** React, Next.js 13, Ant Design v5, graphql-request
- **Database:** PostgreSQL with Flyway migrations
- **Search:** Elasticsearch
- **Messaging:** RabbitMQ
- **Auth:** OIDC/JWT
- **Build:** Gradle (Kotlin DSL), JDK 21 required

---

## Rules

These rules apply unconditionally. Follow them in every change, without exception.

### Database
- **Never** modify existing Flyway migration files — always add a new one
- **Always** add `ON DELETE CASCADE` on FK references to entity tables
- Use `SERIAL PRIMARY KEY NOT NULL` for auto-increment primary keys

### Frontend
- **Never** access `localStorage` directly — always use the wrapper functions in `@components/storage/local`

### Property Types
- **Never** rename a `PropertyType` class after it is deployed — its fully qualified class name (FQCN) is its persistent storage ID

### Workflow
- When fixing a GitHub issue: **always** create a branch named `claude/<short-description>-pipeline` before making any changes (use `/fix-issue` skill)
- **Never** create a pull request unless explicitly asked

### Development Process (TDD)

Follow this order for every non-trivial change:

1. **Write tests first**, then implement:
   - Prefer unit tests (`*Test.kt`) for pure logic
   - Use integration tests (`*IT.kt`) for database/service interactions
   - Use KDSL acceptance tests (`ontrack-kdsl-acceptance/`) for API-level scenarios
   - Use UI tests (`ontrack-web-tests/`) for frontend flows

2. **Implement** the feature/fix to make the tests pass

3. **Run the tests** to verify:

   | Scope               | Gradle task              |
   |---------------------|--------------------------|
   | Unit tests          | `./gradlew test`         |
   | Integration tests   | `./gradlew integrationTest` |
   | KDSL API tests      | `./gradlew kdslAcceptanceTest` |
   | UI tests            | `./gradlew uiTest`       |

### Tests
- **Always** use `Roles.*` constants for role names in tests — never string literals

---

## Build & Run

```bash
# Start dev infrastructure (Postgres, Elasticsearch, RabbitMQ, Keycloak)
./gradlew devComposeUp

# Stop dev infrastructure
./gradlew devComposeDown

# Run all unit tests
./gradlew test

# Run integration tests (requires Docker)
./gradlew integrationTest

# Full build
./gradlew build

# Start backend (after devComposeUp) - use IntelliJ run config "Application (kdsl)"
# Backend runs on http://localhost:8080

# Start frontend - from ontrack-web-core/
npm run dev
# Frontend runs on http://localhost:3000
```

---

## Module Architecture

| Module                                           | Role                                                    |
|--------------------------------------------------|---------------------------------------------------------|
| `ontrack-model`                                  | Domain model, interfaces, events                        |
| `ontrack-repository` / `ontrack-repository-impl` | Database access layer                                   |
| `ontrack-service`                                | Business logic implementations                          |
| `ontrack-ui`                                     | REST controllers, Spring Boot app entry point           |
| `ontrack-ui-graphql`                             | GraphQL schema wiring                                   |
| `ontrack-extension-api`                          | Extension interfaces and base classes                   |
| `ontrack-extension-support`                      | Shared extension utilities                              |
| `ontrack-extension-general`                      | Core built-in features (labels, release property, etc.) |
| `ontrack-extension-{name}`                       | Each external integration (github, jenkins, jira, etc.) |
| `ontrack-database`                               | Flyway migrations                                       |
| `ontrack-web-core`                               | React/Next.js frontend                                  |
| `ontrack-test-utils`                             | Shared test fixtures                                    |
| `ontrack-it-utils`                               | Integration test infrastructure                         |

---

## Backend Development Patterns

### Naming Conventions

| What                    | Pattern                                 | Example                                 |
|-------------------------|-----------------------------------------|-----------------------------------------|
| Package root            | `net.nemerosa.ontrack.extension.{name}` | `net.nemerosa.ontrack.extension.github` |
| GraphQL type class      | `GQLType*`                              | `GQLTypeProject`                        |
| GraphQL root query      | `GQLRootQuery*`                         | `GQLRootQueryBuilds`                    |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nemerosa/ontrack](https://github.com/nemerosa/ontrack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
