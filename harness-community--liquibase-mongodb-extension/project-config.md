---
trigger: always_on
description: This is the Liquibase MongoDB Extension, a Harness-enhanced fork that provides MongoDB support for Liquibase database migrations. The extension allows executing MongoDB operations through Liquibase changesets and includes Harness-specific enhancements:
---

# AGENTS.md - Liquibase MongoDB Extension

## Project Overview

This is the Liquibase MongoDB Extension, a Harness-enhanced fork that provides MongoDB support for Liquibase database migrations. The extension allows executing MongoDB operations through Liquibase changesets and includes Harness-specific enhancements:

- **Mongosh-backed executor** for inline and file-based shell scripts (`mongo` and `mongoFile` changes)
- **executeNative command** for arbitrary MongoDB JSON commands
- **mongoIndexExists precondition** for safer index management

The project extends Liquibase's NoSQL support with MongoDB-specific implementations.

**Repository**: liquibase/liquibase-mongodb  
**Maven Artifact**: `org.liquibase.ext:liquibase-mongodb`  
**Current Version**: 4.33.0.1-SNAPSHOT

## Build System

- **Build tool**: Maven
- **Build project**: `mvn clean install`
- **Build without tests**: `mvn clean install -DskipTests`
- **Package JAR**: `mvn package`
- **Install to local repo**: `mvn install`

## Testing

- **Run all tests**: `mvn test`
- **Run integration tests**: `mvn test -Prun-its`
- **Run specific test class**: `mvn test -Dtest=ClassName`
- **Run single test method**: `mvn test -Dtest=ClassName#methodName`
- **Test file pattern**: `*Test.java` in `src/test/java/`

**MongoDB Connection**: Tests require a MongoDB instance. Connection string is configured in `src/test/resources/liquibase.properties`:
```
url=mongodb://localhost:27017/test_db?socketTimeoutMS=100&connectTimeoutMS=100&serverSelectionTimeoutMS=100
```

**Important**: Adjust the connection string in `liquibase.properties` before running tests if using a different MongoDB instance.

## Linting & Formatting

- **Check style**: Maven parent POM may include Checkstyle or Spotless (check `pom.xml` for plugins)
- **No automatic formatting detected**: No pre-commit hooks found
- **Manual review**: Code style follows standard Java conventions

## Git Workflow

- **Branch naming**: `feature/DBOPS-123-short-description` or `fix/DBOPS-456-short-description`
- **Commit format**: `<type>: [DBOPS-XXX]: <description>`
  - Types: `feat`, `fix`, `chore`, `docs`, `test`, `refactor`, `perf`
  - Example: `feat: [DBOPS-1941]: Add mongoIndexExists precondition`
  - Example: `fix: [DBOPS-2105]: Handle null connection string in getVisibleUrl`
- **PR title format**: Same as commit format
- **Default branch**: `main`

## DOs

- Always run tests before committing (`mvn test`)
- Follow existing code patterns in the codebase
- Use descriptive commit messages with JIRA ticket references (DBOPS-XXX)
- Add unit tests for all new functionality
- Update relevant documentation when adding new features or changes
- Keep changes focused and atomic per commit
- Ensure MongoDB connection is available before running integration tests
- Use the `run-its` Maven profile when running integration tests

## DON'Ts

- Never force push to main branch
- Never commit secrets, credentials, or connection strings with real passwords
- Never skip tests when making functional changes
- Never commit IDE-specific files (.idea/, *.iml, *.iws, .project, .classpath)
- Never modify the Maven parent POM reference without team approval
- Never change the Liquibase core version without verifying compatibility

## Commands to Never Run

- `git push --force origin main`
- `git push --force origin master`
- `git commit --no-verify` (never skip hooks, even if none currently configured)
- `git push --no-verify` (never skip hooks)
- `rm -rf /` or any destructive recursive delete
- `DROP DATABASE` or `DROP TABLE` commands on production databases

## Project Structure

```
liquibase-mongodb/
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   ├── liquibase/
│   │   │   │   ├── ext/mongodb/          # MongoDB-specific Liquibase extensions
│   │   │   │   │   ├── change/           # MongoDB change types (createCollection, insertOne, etc.)
│   │   │   │   │   ├── changelog/        # Changelog parsing and handling
│   │   │   │   │   ├── command/          # Custom commands (executeNative, etc.)
│   │   │   │   │   ├── configuration/    # MongoDB connection configuration
│   │   │   │   │   ├── database/         # MongoDB database implementation
│   │   │   │   │   ├── lockservice/      # Database lock service for MongoDB
│   │   │   │   │   ├── precondition/     # MongoDB preconditions (mongoIndexExists, etc.)
│   │   │   │   │   ├── statement/        # MongoDB statement implementations
│   │   │   │   │   └── tools/            # Utility tools and helpers
│   │   │   │   └── nosql/                # Generic NoSQL infrastructure
│   │   │   │       ├── changelog/        # NoSQL changelog tracking
│   │   │   │       ├── database/         # NoSQL database abstraction
│   │   │   │       ├── executor/         # NoSQL executors (including MongoshExecutor)
│   │   │   │       ├── lockservice/      # NoSQL lock service
│   │   │   │       ├── parser/           # NoSQL changelog parsers
│   │   │   │       ├── snapshot/         # NoSQL snapshot generation
│   │   │   │       └── statement/        # NoSQL statement abstractions

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [harness-community/liquibase-mongodb-extension](https://github.com/harness-community/liquibase-mongodb-extension) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
