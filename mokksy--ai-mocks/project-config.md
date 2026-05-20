---
trigger: always_on
description: Mokksy and AI-Mocks are a suite of tools designed for mocking HTTP and LLM (Large Language Model) APIs for testing and
---

# Mokksy and AI-Mocks Project Guidelines

## Project Overview

Mokksy and AI-Mocks are a suite of tools designed for mocking HTTP and LLM (Large Language Model) APIs for testing and
development purposes.

### Components

1. **Mokksy**

- A mock HTTP server built with Kotlin and Ktor
- Supports true Server-Side Events (SSE) and streaming responses
- Designed to overcome limitations in existing tools like WireMock
- Provides a flexible, fluent Kotlin DSL API

2. **AI-Mocks**

- Specialized mock server implementations built on top of Mokksy
- Currently supports:
  - OpenAI API (`ai-mocks-openai`)
  - Anthropic API (`ai-mocks-anthropic`)
  - Google Gemini (`ai-mocks-gemini`)
- Allows developers to mock LLM API responses for testing and development

### Key Features

- **Streaming Support**: True support for streaming responses and Server-Side Events (SSE)
- **Response Control**: Flexibility to control server responses directly
- **Delay Simulation**: Support for simulating response delays and delays between chunks
- **Modern API**: Fluent Kotlin DSL API with Kotest Assertions
- **Error Simulation**: Ability to mock negative scenarios and error responses

### Project Structure

- `ai-mocks-a2a-client`: Agent2Agent (A2A) client. Used for testing
- `ai-mocks-a2a`: Agent2Agent (A2A) protocol mocks
- `ai-mocks-a2a-models`: Agent2Agent (A2A) protocol models. Copy
  of [A2A specification](ai-mocks-a2a-models/a2a-specification-0.3.0.md)
- `ai-mocks-core`: Core functionality shared across AI-Mocks implementations
- `ai-mocks-openai`: OpenAI API mock implementation
- `ai-mocks-anthropic`: Anthropic API mock implementation
- `ai-mocks-gemini`: Google Gemini/GenAI API mock implementation
- `ai-mocks-ollama`: Ollama API mock implementation
- `mokksy`: The underlying mock HTTP server

## Development Guidelines

- Use `git mv` command when renaming files to preserve git history, if possible
- Never commit or push changes to git automatically. Always confirm.
- Ensure new code follows existing code style and design patterns.
- Use MCP servers, if available, to edit code and run tests. Run terminal commands directly.

### Code Style

#### Kotlin

- Follow Kotlin coding conventions
- Use the provided `.editorconfig` for consistent formatting
- Use Kotlin typesafe DSL builders where possible and prioritize fluent builders style over standard builder methods.
  If DSL builders produce less readable code, use standard setter methods.
- Prefer DSL builder style (method with lambda blocks) over constructors, if possible.
- Use Kotlin's `val` for immutable properties and `var` for mutable properties
- Use fully qualified imports instead of star imports
- Ensure to preserve backward compatibility when making changes

#### Java

- Use the provided `.editorconfig` for consistent formatting
- For Java code, prefer fluent DSL style over standard bean getters and setter methods

### Testing

- Write comprehensive tests for new features
- Prioritize test readability
- Use function `Names with backticks` for test methods in Kotlin, e.g. "fun `should return 200 OK`()"
- Avoid writing KDocs for tests, keep code self-documenting
- Write Kotlin tests with [kotlin-test](https://github.com/JetBrains/kotlin/tree/master/libraries/kotlin.test),
  [mockk](https://mockk.io/) and [Kotest-assertions](https://kotest.io/docs/assertions/assertions.html)
  with infix form assertions `shouldBe` instead of `assertEquals`.
- Use Kotest's `withClue("<failure reason>")` to describe failure reasons, but only when the assertion is NOT obvious.
  Remove obvious cases for simplicity.
- If multiple assertions exist against nullable field, first check for null, e.g.:
  `params shouldNotBeNull { params.id shouldBe 1 }`
- For testing json serialization use [Kotest-assertions-json](https://kotest.io/docs/assertions/json/json-overview.html)
  assertions, e.g. `shouldEqualJson` and never compare substrings.
- Prefer `assertSoftly(subject) { }` over `assertSoftly { }`. Use the no-subject form only when asserting 
  the same invariant across multiple related objects, or when the assertions are tightly coupled.
- When asked to write tests in Java: use JUnit6, Mockito, AssertJ core

#### Parallel Test Execution and Shared Mock Servers

- Apply the `integration-testing` skill when writing or modifying integration tests.
- Tests **must** be safe to run in parallel. Do not disable parallel execution as a workaround for test failures.
- All tests within a module share a single top-level mock server instance. Do not create per-test or per-class isolated mock instances unless specifically testing negative scenarios.
- Mock stubs are **immutable** once registered.
- Use randomized but distinct values for `model`, `seed`, `temperature`, `topK`, `topP`, `maxOutputTokens`, etc. in `@BeforeEach` to ensure parallel safety.
- For each LLM use their native SDK as primary test client. Add smoke-level tests using LangChain4j and SpringAI.

### Documentation

- Update README files when adding new features
- Document API changes in the appropriate module's documentation
- Write tutorials in Hugo markdown /docs/content/docs

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mokksy/ai-mocks](https://github.com/mokksy/ai-mocks) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
