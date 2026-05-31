---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a collection of Spring AI examples demonstrating various AI integration patterns and use cases. The repository is organized as a multi-module Maven project with examples covering:

- **Agentic Patterns**: Chain workflows, orchestrator-workers, parallelization, routing, and evaluator-optimizer patterns
- **Model Context Protocol (MCP)**: Extensive examples for MCP clients and servers including SQLite, filesystem, weather, web search, and annotations
- **Kotlin Examples**: Hello world, function callbacks, and RAG implementations in Kotlin
- **Miscellaneous**: Function callbacks, streaming responses, and prompt engineering patterns

## Build System and Commands

This project uses Maven with the Maven Wrapper (mvnw). Each module can be built and run independently.

### Common Development Commands

**Build entire project:**
```bash
./mvnw clean package
```

**Run a specific example:**
```bash
./run-example.sh <project-directory-name>
# Example: ./run-example.sh agentic-patterns/chain-workflow
```

**Build and run individual module:**
```bash
cd <module-directory>
./mvnw clean package
./mvnw spring-boot:run
```

**Build from root (all modules):**
```bash
mvn clean package
```

### Version Management

**Update Spring AI version across all modules:**
```bash
./scripts/update-spring-ai-version.sh 1.0.1  # or 1.1.0-SNAPSHOT
```

The version management script handles two patterns:
- **17 modules** use `<spring-ai.version>` property
- **15 modules** use direct BOM version in dependencyManagement
- **Total: 32 pom.xml files** updated across the repository

**Check current versions:**
```bash
./scripts/check-spring-ai-version.sh
```

**Restore from backup:**
```bash
./scripts/restore-spring-ai-version.sh /path/to/backup
```

## Architecture

### Module Structure
- Each example is a self-contained Spring Boot application
- Modules follow standard Maven directory structure: `src/main/java` and `src/test/java`
- Common Spring AI version: `1.1.0-SNAPSHOT`
- Java version: 17
- Spring Boot parent: `3.4.5`

### Key Module Categories

**Agentic Patterns (`agentic-patterns/`):**
- Demonstrates AI agent workflow patterns
- Each pattern is a separate module with its own Application class

**Model Context Protocol (`model-context-protocol/`):**
- MCP client and server implementations
- Covers various protocols: SQLite, filesystem access, weather APIs, web search
- Includes both manual and starter-based implementations
- Server types: WebMVC, WebFlux, and STDIO

**Kotlin Examples (`kotlin/`):**
- Kotlin-based Spring AI implementations
- Includes RAG (Retrieval-Augmented Generation) examples

### Configuration Patterns
- Application properties typically in `src/main/resources/application.properties` or `application.yaml`
- MCP configurations often use `mcp-servers-config.json`
- Docker Compose files for complex setups (e.g., `compose.yml`)

## Testing Framework

This repository uses a comprehensive integration testing framework with **AI-powered validation** for ensuring all examples work correctly across releases. The framework currently covers **24 modules with integration tests** with intelligent validation for interactive applications.

### Testing Approaches

| Example Type | Test Location | Command | Purpose |
|--------------|---------------|---------|---------|
| **Simple** | `src/test/java/` | `./mvnw test` | Unit tests, basic functionality |
| **Complex** | `integration-tests/` | `jbang integration-tests/Run*.java` | End-to-end integration with AI validation |

### Key Features

- **24 Modules Tested**: Integration tests for all major example categories
- **AI Validation**: Intelligent analysis of application logs for non-deterministic outputs
- **Interactive Application Support**: Testing of chatbots and Scanner-based applications
- **Centralized Architecture**: Single source of truth with 84% code reduction
- **Comprehensive Logging**: Full application output preserved for debugging

### Primary Integration Testing Tools

The framework provides **two essential tools** for integration testing:

**1. ⭐ Run all integration tests (primary test runner):**
```bash
./integration-testing/scripts/run-integration-tests.sh
```

**Run specific integration test:**
```bash
./integration-testing/scripts/run-integration-tests.sh module-name
```

**Run with clean logs:**
```bash
./integration-testing/scripts/run-integration-tests.sh --clean-logs
```

**2. ⭐ Create integration tests for new examples (scaffolding tool):**
```bash
# Essential for extending the framework - creates ExampleInfo.json + Run*.java files
python3 integration-testing/scripts/scaffold_integration_test.py <module-path> [--complexity simple|complex|mcp]
```

**Create integration test with AI validation:**
```bash
# Simple example (hybrid AI validation)
python3 integration-testing/scripts/scaffold_integration_test.py kotlin/kotlin-hello-world --complexity simple

# Complex workflow (primary AI validation)
python3 integration-testing/scripts/scaffold_integration_test.py agentic-patterns/chain-workflow --complexity complex

# Interactive application (fallback AI validation)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [spring-projects/spring-ai-examples](https://github.com/spring-projects/spring-ai-examples) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-31 -->
