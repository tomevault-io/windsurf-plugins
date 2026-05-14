---
trigger: always_on
description: This guide helps Claude Code (and contributors using it) work effectively with the Quarkus Flow project.
---

# Quarkus Flow - Claude Code Guide

This guide helps Claude Code (and contributors using it) work effectively with the Quarkus Flow project.

## ⚠️ CRITICAL: Required Workflow Validation

**BEFORE CREATING ANY PULL REQUEST**, you MUST run the full build with integration tests:

```bash
./mvnw clean install -DskipITs=false
```

This is **NON-NEGOTIABLE**. This command ensures:
- All unit tests pass
- All integration tests pass (including cross-module compatibility)
- No build errors across all modules
- Dev Services (Testcontainers) work correctly

**For Claude Code**: When the user asks you to create a PR, you MUST run this command first and verify it succeeds before proceeding with `gh pr create`. If the build fails, fix the issues before creating the PR.

## Project Overview

**Quarkus Flow** is a lightweight workflow engine for Quarkus based on the CNCF Serverless Workflow specification. It supports classic workflows and Agentic AI orchestrations with LangChain4j integration.

- **Tech Stack**: Java 17+, Maven, Quarkus framework
- **Architecture**: Multi-module Quarkus extension with runtime/deployment split
- **Spec**: CNCF Serverless Workflow (https://serverlessworkflow.io/)
- **Docs**: https://docs.quarkiverse.io/quarkus-flow/dev/

## Repository Structure

```
quarkus-flow/
├── core/                    # Core workflow engine
│   ├── runtime/            # Runtime code
│   ├── deployment/         # Build-time/deployment code
│   ├── runtime-dev/        # Dev mode features
│   └── integration-tests/  # Integration tests
├── messaging/              # Reactive Messaging integration
├── langchain4j/           # LangChain4j integration for agentic workflows
├── persistence/           # Workflow persistence support
├── durable-kubernetes/    # Kubernetes-native durable execution
├── scheduler/             # Scheduled workflow execution
├── docs/                  # Antora documentation
├── examples/              # Example applications
└── bom/                   # Bill of Materials for dependency management
```

## Build & Test Commands

### Standard build (includes unit tests)
```bash
./mvnw clean install
```
This automatically runs unit tests. Integration tests are skipped by default.

### Quick build (skip all tests)
```bash
./mvnw clean install -DskipTests
```

### Build specific module
```bash
./mvnw clean install -pl core -am  # -am = also make dependencies
```

### Run documentation locally
```bash
./mvnw -pl docs -am quarkus:dev
# Press 'w' when Quarkus starts to open the docs site
```

### Pre-PR validation (REQUIRED)
```bash
./mvnw clean install -DskipITs=false
```

**Why this matters**:
- Integration tests validate cross-module compatibility
- Catches issues with Quarkus Dev Services and Testcontainers
- Ensures all modules work together correctly
- Prevents CI failures and broken builds
- Required by project policy before any PR creation

**When to run**: Before creating ANY pull request. No exceptions.

## Testing

- **Unit tests**: Run via Surefire (`**/src/test/**/*Test.java`) - automatically included in `./mvnw clean install`
- **Integration tests**: Run via Failsafe (`**/src/test/**/*IT.java`) - require `-DskipITs=false`
- **Test matrix**: Ubuntu + Windows, JDK 17/21/25
- Integration tests use Quarkus Dev Services (Testcontainers)

### Important Testing Rules

**Mocked LLM Calls**: Integration tests mock Ollama/LLM model calls to avoid resource-intensive operations in CI. Never make real LLM API calls in tests.

**Parallel Execution**: Tests run in parallel. **Never use fixed ports** (e.g., 8080). Use unusual/random ports or let Quarkus assign them automatically.

**Special Test Profiles**: Some tests require optional dependencies that are isolated with Maven profiles to avoid affecting other tests:

```bash
# Test quarkus-logging-json integration (adds dependency only for this test)
./mvnw test -pl core/integration-tests -Ptest-logging-json -Dtest=StructuredLoggingWithQuarkusLoggingJsonTest
```

When writing tests:
- Use AssertJ for assertions (preferred in this project)
- Follow existing test patterns in each module
- Integration tests go in `integration-tests/` submodules
- **Examples tests**: Mock using `quarkus-mockito`
- **Integration tests**: Mock using `WireMock` for external services
- **Test naming**: Use `snake_case` with `@DisplayName` annotation (e.g., `@DisplayName("test_workflow_execution_completes")`)
- Mock external services (LLMs, APIs) to keep tests fast and reliable

## Code Conventions

### Quarkus Extension Pattern
Each module follows Quarkus extension structure:
- `runtime/`: Code that runs in the application
- `deployment/`: Build-time processors, code generation
- Never reference deployment code from runtime code

### CDI & Build Items
- Use `@BuildStep` in deployment modules for build-time processing
- Runtime beans use standard CDI annotations (`@ApplicationScoped`, etc.)
- Build items are the contract between build steps

### Serverless Workflow DSL
- Workflows extend `io.quarkiverse.flow.Flow`
- Use the fluent DSL from `io.serverlessworkflow.fluent.func.dsl.FuncDSL`
- Support both Java DSL and YAML workflow definitions

## Documentation

Documentation uses **Antora** format:
- Source: `docs/modules/ROOT/`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [quarkiverse/quarkus-flow](https://github.com/quarkiverse/quarkus-flow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
