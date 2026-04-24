---
trigger: always_on
description: A Java SDK for programmatic control of GitHub Copilot CLI. This is a port of the official .NET SDK, targeting Java 17+.
---

# Copilot Instructions for copilot-sdk-java

A Java SDK for programmatic control of GitHub Copilot CLI. This is a port of the official .NET SDK, targeting Java 17+.

## About These Instructions

These instructions guide GitHub Copilot when assisting with this repository. They cover:

- **Tech Stack**: Java 17+, Maven, Jackson for JSON, JUnit for testing
- **Purpose**: Provide a Java SDK for programmatic control of GitHub Copilot CLI
- **Architecture**: JSON-RPC client communicating with Copilot CLI over stdio
- **Key Goals**: Maintain parity with reference implementation .NET SDK while following Java idioms

## Build & Test Commands

```bash
# Build and run all tests
mvn clean verify

# Run a single test class
mvn test -Dtest=CopilotClientTest

# Run a single test method
mvn test -Dtest=ToolsTest#testToolInvocation

# Format code (required before commit)
mvn spotless:apply

# Check formatting only
mvn spotless:check

# Build without tests
mvn clean package -DskipTests

# Run tests with debug logging
mvn test -Pdebug
```

### Running Tests from AI Agents / Copilot

When running tests to verify changes, **always use `mvn verify` without `-q` and without piping through `grep`**. The full output is needed to diagnose failures. Do NOT use commands like:

```bash
# BAD - hides critical failure details, often requires a second run
mvn verify -q 2>&1 | grep -E 'Tests run:|BUILD'
mvn verify 2>&1 | grep -E 'Tests run.*in com|BUILD|test failure'
```

Instead, use one of these approaches:

```bash
# GOOD - run tests with full output (preferred for investigating failures)
mvn verify

# GOOD - run tests showing just the summary and result using Maven's built-in log level
mvn verify -B --fail-at-end 2>&1 | tail -30

# GOOD - run a single test class when debugging a specific test
mvn test -Dtest=CopilotClientTest
```

**Interpreting results:**
- `BUILD SUCCESS` at the end means all tests passed. No further investigation needed.
- `BUILD FAILURE` means something failed. The lines immediately above `BUILD FAILURE` will contain the relevant error information. Look for `[ERROR]` lines near the bottom of the output.
- The Surefire summary line `Tests run: X, Failures: Y, Errors: Z, Skipped: W` appears near the end and gives the counts.
- **Do NOT run tests a second time** just to get different output formatting. One run with full output is sufficient.

## Architecture

### Core Components

- **CopilotClient** - Main entry point. Manages connection to Copilot CLI server via JSON-RPC over stdio. Spawns CLI process or connects to existing server.
- **CopilotSession** - Represents a conversation session. Handles event subscriptions, tool registration, permissions, and message sending.
- **JsonRpcClient** - Low-level JSON-RPC protocol implementation using Jackson for serialization.

### Package Structure

- `com.github.copilot.sdk` - Core classes (CopilotClient, CopilotSession, JsonRpcClient)
- `com.github.copilot.sdk.json` - DTOs, request/response types, handler interfaces (SessionConfig, MessageOptions, ToolDefinition, etc.)
- `com.github.copilot.sdk.generated` - Generated event types for session streaming (SessionEvent, AssistantMessageEvent, SessionIdleEvent, ToolExecutionStartEvent, etc.)

### Test Infrastructure

Tests use the official copilot-sdk test harness from `https://github.com/github/copilot-sdk`. The harness is automatically cloned during `generate-test-resources` phase to `target/copilot-sdk/`.

- **E2ETestContext** - Manages test environment with CapiProxy for deterministic API responses
- **CapiProxy** - Node.js-based replaying proxy using YAML snapshots from `test/snapshots/`
- Test snapshots are stored in the reference implementation repo's `test/snapshots/` directory

## Key Conventions

### Reference Implementation Merging

This SDK tracks the official .NET implementation at `github/copilot-sdk`. The `.lastmerge` file contains the last merged reference implementation commit hash. Use the `agentic-merge-reference-impl` skill (see `.github/prompts/agentic-merge-reference-impl.prompt.md`) to port changes.

When porting from .NET:
- Adapt to Java idioms, don't copy C# patterns directly
- Convert `async/await` → `CompletableFuture`
- Convert C# properties → Java getters/setters or fluent setters
- Use Jackson for JSON (`ObjectMapper`, `@JsonProperty`)

### Code Style

- 4-space indentation (enforced by Spotless with Eclipse formatter)
- Fluent setter pattern for configuration classes (e.g., `new SessionConfig().setModel("gpt-5").setTools(tools)`)
- Public APIs require Javadoc (enforced by Checkstyle, except `json` and `events` packages)
- Pre-commit hook runs `mvn spotless:check` - Must be manually enabled with: `git config core.hooksPath .githooks`, except in the Copilot coding agent environment. This hook is explicitly enabled in the Copilot coding agent environment. See [copilot-setup-steps.yml](workflows/copilot-setup-steps.yml).

### Handler Pattern

Handlers use functional interfaces with `CompletableFuture` returns:

```java
session.createSession(new SessionConfig()
    .setOnPermissionRequest((request, invocation) -> 
        CompletableFuture.completedFuture(new PermissionRequestResult().setKind("allow")))

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [github/copilot-sdk-java](https://github.com/github/copilot-sdk-java) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
