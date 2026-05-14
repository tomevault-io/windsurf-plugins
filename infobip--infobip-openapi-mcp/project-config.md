---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**infobip-openapi-mcp** is an open-source Java framework for exposing OpenAPI-documented HTTP APIs as MCP (Model Context
Protocol) servers. It is built on Spring AI and Spring Boot, and is published to Maven Central.

## Build & Development Commands

Project uses Java 21. Make sure that local tooling supports this version. **DO NOT** attempt to downgrade the source
code to earlier Java versions. Look for newer JKD instead, for example using the SDKMAN version manager.

```bash
# Build the project (runs tests + spotless format check)
mvn verify

# Build skipping tests
mvn package -DskipTests

# Run all tests
mvn test

# Run tests for a specific module
mvn test -pl infobip-openapi-mcp-core
mvn test -pl infobip-openapi-mcp-spring-boot-starter -am

# Run a single test class
mvn test -pl infobip-openapi-mcp-core -Dtest=DiscriminatorFlattenerTest
mvn test -pl infobip-openapi-mcp-spring-boot-starter -am -Dtest=ToolCallIntegrationTest

# Apply code formatting (Palantir Java Format via Spotless)
mvn spotless:apply

# Check formatting without applying
mvn spotless:check

# Install git hooks (runs spotless:apply on pre-commit)
mvn install -Pgit-hook

# Check version of Java that is used by maven
mvn --version

# Check available Java versions with sdkman
PAGER=cat sdk list java | grep -E 'installed|local'

# Pick identifier of installed Java version equal or greater than 21 (for example 25-tem) and enable it with sdkman
sdk use java <identifier>
```

**ALWAYS** do this after completing any coding task:

- Run `mvn spotless:apply` as the final step before presenting results.
- Update `CHANGELOG.md` under the `[Unreleased]` section using
  the [Keep a Changelog](https://keepachangelog.com/en/1.1.0/) format (`Added`, `Changed`,  `Fixed`, `Removed`).
  Write entries from a user perspective — describe the feature and its value, not the classes or internal mechanics
  behind it. It is fine to mention configuration properties needed to enable or customize a feature, or java interfaces
  that users can implement such as `OpenApiFilter` or `ApiRequestEnricher`. Avoid class names, method names, test names,
  and other implementation details.
- If you added or changed an external configuration property, add or update its row in the properties table in
  `README.md`.
- Check and update `CLAUDE.md` to reflect the new state of the project.

## Module Structure

This is a two-module Maven project:

- **`infobip-openapi-mcp-core`** — Framework core logic. No Spring Boot autoconfiguration, suitable as a library
  dependency.
- **`infobip-openapi-mcp-spring-boot-starter`** — Spring Boot autoconfiguration that wires the core beans. This is what
  users add to their `pom.xml`.

## Architecture

The framework follows this startup flow:

1. `OpenApiRegistry` reads and caches the OpenAPI spec from `infobip.openapi.mcp.open-api-url`
2. `OpenApiFilterChain` applies `OpenApiFilter` beans (e.g., `DiscriminatorFlattener`, `PatternPropertyRemover`) to
   transform the spec
3. `ToolRegistry` converts each API operation into a `RegisteredTool` using `InputSchemaComposer`,
   `InputExampleComposer`, `ToolAnnotationResolver`, and the configured `NamingStrategy`
4. Tools are registered with the Spring AI MCP server (SSE, Streamable HTTP, Stateless HTTP, or stdio transport)

**Runtime tool call flow:**
`ToolSpecBuilder` → `ToolCallFilterChain` (ordered `ToolCallFilter` beans) → `RegisteredTool` (lowest precedence, makes
HTTP call via `ToolHandler`) → optional `JsonDoubleSerializationCorrector` retry logic

### Key Extension Points

| Interface            | Purpose                                                                                                                  |
|----------------------|--------------------------------------------------------------------------------------------------------------------------|
| `OpenApiFilter`           | Transform the OpenAPI spec before tool metadata is built; disable via `infobip.openapi.mcp.filters.[filter-name]: false` |
| `ApiRequestEnricher`      | Modify HTTP requests to the downstream API (headers, metadata); failures are swallowed                                   |
| `ToolCallFilter`          | Intercept tool calls; can abort the chain unlike enrichers                                                               |
| `NamingStrategy`          | Custom tool name generation; replace the default bean                                                                    |
| `ErrorModelProvider`      | Custom error response format returned to MCP clients                                                                     |
| `CredentialProvider`      | Supply credentials from any source (HTTP header, vault, env, etc.); replace the default bean                             |

Important: filters, enrichers, strategies and providers can be implemented by application code, which is outside the
framework. You will not see those implementations in this project's source code. This is the supported way to extend and

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [infobip/infobip-openapi-mcp](https://github.com/infobip/infobip-openapi-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
