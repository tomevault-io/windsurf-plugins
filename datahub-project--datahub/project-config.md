---
trigger: always_on
description: This is the canonical reference for working with the DataHub codebase. It applies to all coding
---

# DataHub — Agent Development Guide

This is the canonical reference for working with the DataHub codebase. It applies to all coding
agents (Claude Code, Cursor, Codex CLI, Devin, etc.) and human developers alike.

## Code Navigation (LSP)

Prefer LSP tools over Grep for code navigation tasks:

- Use `goToDefinition` to find where something is defined
- Use `findReferences` to find all call sites
- Use `workspaceSymbol` to find symbols by name
- Use diagnostics after any edit to catch type errors immediately

See [docs/lsp-setup.md](docs/lsp-setup.md) for installation and configuration.

## Essential Commands

**Build and test:**

```bash
./gradlew build           # Build entire project
./gradlew check           # Run all tests and linting
./gradlew format          # Format all code (Java, Markdown, GraphQL, YAML)

# Note that each directory typically has a build.gradle file, but the available tasks follow similar conventions.

# Java code.
./gradlew spotlessApply   # Java code formatting

# Python code.
./gradlew :metadata-ingestion:testQuick     # Fast Python unit tests
./gradlew :metadata-ingestion:lint          # Python linting (ruff, mypy)
./gradlew :metadata-ingestion:lintFix       # Python linting auto-fix (ruff only)

# Markdown, GraphQL, YAML formatting
./gradlew :datahub-web-react:mdPrettierWrite        # Format markdown files
./gradlew :datahub-web-react:graphqlPrettierWrite   # Format GraphQL schemas
./gradlew :datahub-web-react:githubActionsPrettierWrite # Format GitHub Actions
```

**IMPORTANT: Verifying Python code changes:**

- **ALWAYS use `./gradlew :metadata-ingestion:lintFix`** to verify Python code changes
- **NEVER use `python3 -m py_compile`** - it doesn't catch style issues or type errors
- **NEVER use `ruff` or `mypy` commands directly** - use the Gradle task instead
- lintFix runs ruff formatting and fixing automatically, ensuring code quality
- For smoke-test changes, the lintFix command will also check those files

## Code Formatting and Linting

**CRITICAL: Always use Gradle tasks for formatting and linting. Never use npm/yarn/npx commands directly.**

### Available Formatting Tasks

**Format everything:**

```bash
./gradlew format              # Format all code (Java, Markdown, GraphQL, YAML)
./gradlew formatChanged       # Format only changed files (faster)
```

**Format specific file types:**

```bash
# Markdown files
./gradlew :datahub-web-react:mdPrettierWrite        # Format all markdown
./gradlew :datahub-web-react:mdPrettierCheck        # Check markdown formatting

# GraphQL schemas
./gradlew :datahub-web-react:graphqlPrettierWrite   # Format GraphQL files
./gradlew :datahub-web-react:graphqlPrettierCheck   # Check GraphQL formatting

# GitHub Actions YAML
./gradlew :datahub-web-react:githubActionsPrettierWrite   # Format workflow files
./gradlew :datahub-web-react:githubActionsPrettierCheck   # Check workflow files

# Java code
./gradlew spotlessApply       # Format Java code

# Python code
./gradlew :metadata-ingestion:lintFix      # Format and fix Python code
./gradlew :metadata-ingestion:lint         # Check Python formatting
```

### When CI Formatting Checks Fail

If you see CI failures like:

- `markdown_format / markdown_format_check (pull_request)` - Use `./gradlew :datahub-web-react:mdPrettierWrite`
- `graphql_prettier_check` - Use `./gradlew :datahub-web-react:graphqlPrettierWrite`
- `spotlessJavaCheck` - Use `./gradlew spotlessApply`
- Python linting failures - Use `./gradlew :metadata-ingestion:lintFix`

**Never do this:**

```bash
npx prettier --write "docs/**/*.md"    # WRONG - bypasses Gradle
yarn prettier --write                   # WRONG - bypasses Gradle
npm run format                          # WRONG - bypasses Gradle
```

**Always do this:**

```bash
./gradlew :datahub-web-react:mdPrettierWrite      # CORRECT - uses Gradle
./gradlew format                                   # CORRECT - formats everything
```

### Why Use Gradle Tasks?

1. **Consistent configuration**: Gradle tasks use the project's Prettier config
2. **Pre-commit hook integration**: Gradle tasks match what CI runs
3. **Dependency management**: Ensures correct tool versions
4. **Cross-platform**: Works reliably across all environments

**Java SDK v2 integration tests:**

See [metadata-integration/java/datahub-client/CLAUDE.md](metadata-integration/java/datahub-client/CLAUDE.md) for detailed integration test documentation.

## Architecture Overview

DataHub is a **schema-first, event-driven metadata platform** with three core layers:

### Core Services

- **GMS (Generalized Metadata Service)**: Java/Spring backend handling metadata storage and REST/GraphQL APIs
- **Frontend**: React/TypeScript application consuming GraphQL APIs
- **Ingestion Framework**: Python CLI and connectors for extracting metadata from data sources
- **Event Streaming**: Kafka-based real-time metadata change propagation

### Key Modules

- `metadata-models/`: Avro/PDL schemas defining the metadata model
- `metadata-service/`: Backend services, APIs, and business logic
- `datahub-web-react/`: Frontend React application
- `metadata-ingestion/`: Python ingestion framework and CLI
- `datahub-graphql-core/`: GraphQL schema and resolvers


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [datahub-project/datahub](https://github.com/datahub-project/datahub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
