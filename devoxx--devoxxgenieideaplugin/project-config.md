---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

DevoxxGenie is a Java-based LLM Code Assistant plugin for IntelliJ IDEA that integrates with both local LLM providers (
Ollama, LMStudio, GPT4All, Llama.cpp, Exo) and cloud-based LLMs (OpenAI, Anthropic, Mistral, Groq, Gemini, DeepInfra,
DeepSeek, OpenRouter, Azure OpenAI, Amazon Bedrock). The plugin supports advanced features like RAG (Retrieval-Augmented
Generation), MCP (Model Context Protocol) servers, web search, and agentic programming capabilities.

## Best Practices

Add at the top of CLAUDE.md under ## Project Context\n\nThis is a JetBrains/IntelliJ plugin project (DevoxxGenie).
Primary language is Java with some TypeScript. Be aware of EDT (Event Dispatch Thread) constraints — UI operations must
run on EDT, and long-running operations must NOT block EDT. Use `ApplicationManager.getApplication().invokeLater()` or
`ReadAction`/`WriteAction` as appropriate.
¬
Add under a top-level ## Workflow Rules section in CLAUDE.md\n\nWhen asked to investigate or fix an issue, do NOT deeply
explore the entire codebase autonomously. Start focused on the specific area mentioned, and ask before expanding scope.
Avoid unnecessary web searches unless explicitly requested.

Add under ## Git Workflow section in CLAUDE.md\n\nAlways create a feature/fix branch BEFORE making any code changes.
Never edit code on the current branch without confirming the branch strategy first.

Add under ## Testing section in CLAUDE.md\n\nWhen asked to investigate a bug, write a reproducing test FIRST before
applying any fix, unless told otherwise.

Add under ## Release Process section in CLAUDE.md\n\nFor version bumps, always ask the user what the target version
should be. Do not assume the next version number.

Create a branch fix/issue-XXX from develop, investigate the bug described in issue #XXX, write a reproducing test first,
then implement the fix, run all tests, and create a PR.

Investigate GitHub issue #[NUMBER]. First, read the issue and explore the relevant code. Then write a minimal failing
test that reproduces the exact bug. Run the test to confirm it fails. Now implement the fix. Run the full test suite
repeatedly, iterating on your implementation until ALL tests pass — including your new regression test. Do not ask me
for feedback until you have a green test suite. Then create a feature branch, commit everything with a descriptive
message, and show me a summary of what you changed and why.

## Build & Development Commands

### Java Version Requirement

**IMPORTANT**: This project requires **JDK 21** for building. JDK 25 causes Gradle build script failures. Always set `JAVA_HOME` before running Gradle commands:

```bash
export JAVA_HOME=~/.sdkman/candidates/java/21-zulu
```

### Building

```bash
./gradlew buildPlugin              # Build plugin (creates ZIP in build/distributions/)
./gradlew clean                    # Clean build artifacts
./gradlew shadowJar                # Create shadow JAR with dependencies
```

### Testing

**IMPORTANT**: Always pipe test output to grep for failures so you can immediately focus on what failed:

```bash
./gradlew test 2>&1 | grep -E "FAILED|failed"
./gradlew test --tests ClassName 2>&1 | grep -E "FAILED|failed"
./gradlew test --tests ClassName.methodName 2>&1 | grep -E "FAILED|failed"
./gradlew verifyPlugin                            # Verify plugin (includes tests)
```

### Running & Publishing

```bash
./gradlew runIde                   # Run IntelliJ IDEA with plugin for testing
./gradlew publishPlugin            # Publish to JetBrains Marketplace (requires PUBLISH_TOKEN env var)
```

### Task Automation (using Taskfile)

```bash
task build                         # Build the plugin
task test                          # Run tests
task run-ide                       # Run IDE with plugin
task generate-changelog VERSION=0.8.0  # Generate changelog from merged PRs
task preview-changes VERSION=0.8.0     # Preview changelog without committing
```

## Core Architecture

### Multi-Module Structure

- **Root module**: Main IntelliJ plugin code (`src/main/java/com/devoxx/genie/`)
- **Core module**: Shared utilities being refactored (see `core/README.md` - issue #564)
- **Docusaurus**: Documentation website (`docusaurus/`)

### Key Architectural Components

#### 1. Prompt Execution Flow

The plugin processes user prompts through a layered architecture:

**Entry Point**:

- `UserPromptPanel` → `PromptSubmissionListener.onPromptSubmitted()` →
  `PromptExecutionController.handlePromptSubmission()`

**Processing Layer**:

- `PromptExecutionService.executeQuery()` - Handles token calculations, RAG, and GitDiff settings
- `ChatPromptExecutor.executePrompt()` - Dispatches to appropriate LLM provider
- `LLMProviderService.getAvailableModelProviders()` - Retrieves model from ChatModelFactory

**Execution Strategies**:

- `StreamingPromptExecutor` - Token-by-token streaming responses
- `NonStreamingPromptExecutionService` - Full response mode
- `WebSearchPromptExecutionService` - Web search augmented prompts

**Response Rendering**:

- `ChatStreamingResponsePanel` - Real-time streaming UI updates

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [devoxx/DevoxxGenieIDEAPlugin](https://github.com/devoxx/DevoxxGenieIDEAPlugin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
