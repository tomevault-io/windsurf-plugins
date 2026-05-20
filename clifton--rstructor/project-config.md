---
trigger: always_on
description: - Always make sure there are relevant tests created and that they pass before finishing each task.
---

# CLAUDE.md - Agent Guidelines for rstructor

## Rules
- Always make sure there are relevant tests created and that they pass before finishing each task.
- Ensure the generated rustdocs are high quality with examples.
- Doctests should not be ignored and should always be checked.
- Do not add logic into examples that should be library features. Make the library feature instead.
- Always assume that test environment has valid LLM provider api keys.
- **Temporary Files**: Place all ad-hoc scripts, temporary markdown files, and other ephemeral files in the `tmp/` directory. This directory is excluded from both git and cargo packaging.

## Build & Test Commands
- Build: `cargo build`
- Run all tests: `cargo test`
- Run specific test: `cargo test test_name`
- Run examples: `cargo run --example example_name`
- Format code: `cargo fmt`
- Lint: `cargo clippy`

## Code Style Guidelines
- Use Rust 2024 edition features
- Follow Rust naming conventions: snake_case for functions/variables, PascalCase for types
- Use clear, descriptive variable and function names
- Document all public API items with rustdoc comments
- Organize imports in order: std, external crates, local modules
- Prefer returning Result over unwrap/expect in public APIs
- Use tracing for logging instead of println
- Use async/await for all network operations
- Follow the builder pattern for configuration objects
- Use thiserror for custom error types
- Test all new functionality with unit tests
- Format code with rustfmt and address clippy warnings before committing
- Use feature flags for optional dependencies (e.g., specific LLM backends)

## Macro Attribute Implementation Guidelines
- Always implement attribute parsing directly within the macro system, not with JSON strings
- Use native Rust syntax for attribute values and complex data structures
- For example, array literals should be parsed directly as `#[llm(examples = ["one", "two", "three"])]`
- Never use JSON serialized strings for attribute values (e.g., do not use `examples = r#"["value1"]"#`)
- Parse container attributes individually without relying on JSON parsing
- Support multiple attribute specification styles that feel natural in Rust
- For multi-value attributes, support both parentheses and array syntax

## Model List Maintenance Guidelines

**CRITICAL**: Model lists must be kept up-to-date with provider releases. When adding new providers or updating existing ones, always refer to the official documentation sources below:

### Official Model Documentation Sources

**OpenAI:**
- Model Documentation: https://platform.openai.com/docs/models
- Models API Endpoint: https://platform.openai.com/docs/api-reference/models/list

**Anthropic:**
- All Models Overview: https://docs.anthropic.com/en/docs/about-claude/models/all-models
- Models List API: https://docs.anthropic.com/en/api/models-list

**xAI (Grok):**
- Models Documentation: https://docs.x.ai/docs/models

**Google (Gemini):**
- Models Documentation: https://ai.google.dev/models
- Models API Endpoint: `GET https://generativelanguage.googleapis.com/v1beta/models?key=$GEMINI_API_KEY`

### Update Process

**CRITICAL RULES:**
1. **ALWAYS use API endpoints to get current models** - Never guess or rely on web search. Use the official API endpoints:
   - **OpenAI**: `GET https://api.openai.com/v1/models` (requires `Authorization: Bearer $OPENAI_API_KEY`)
   - **Anthropic**: `GET https://api.anthropic.com/v1/models` (requires `x-api-key: $ANTHROPIC_API_KEY` and `anthropic-version: 2023-06-01`)
   - **xAI (Grok)**: Check `https://docs.x.ai/docs/models` or use their API if available
   - **Google (Gemini)**: `GET https://generativelanguage.googleapis.com/v1beta/models?key=$GEMINI_API_KEY` (API key as query parameter)
2. **NEVER guess model identifiers** - Always get exact model names from API responses or official documentation
3. **NEVER rely on web search results** - Web search often returns outdated, incorrect, or speculative information
4. **ALWAYS use exact API identifiers** - Model identifiers must match exactly what the API expects (e.g., `gpt-5-chat-latest`, `claude-sonnet-4-5-20250929`, `grok-4-0709`, `gemini-1.5-flash`)
5. **If API key is not available**: Ask the user to provide the exact model identifiers from the API endpoint response, rather than guessing or using web search
6. **NEVER use web search for model lists** - Web search results are often outdated, incorrect, or speculative. Always use official API endpoints or documentation
7. **Verify date stamps make sense** - If version X.Y is newer than X.Z, its date stamp should be later (e.g., Claude 3.7 date should be after Claude 3.5 date)
8. **Check for new major versions** - Don't assume only minor version updates; check for major version releases (e.g., Claude 4, GPT-5, Gemini 2.0)
9. **Verify model name format** - Different providers may use different naming conventions:
   - OpenAI: `gpt-4-turbo`, `gpt-5-chat-latest`
   - Anthropic: `claude-sonnet-4-20250514` (includes date)
   - Grok: `grok-4-0709` (includes date)
   - Gemini: `gemini-1.5-flash`, `gemini-2.0-flash` (may include version and suffix)
10. **Filter for chat completion models** - Only include models suitable for chat completions:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [clifton/rstructor](https://github.com/clifton/rstructor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
