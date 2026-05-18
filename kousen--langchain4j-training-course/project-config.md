---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a **hands-on training course** for learning LangChain4j through progressive lab exercises. The repository is structured as a proper training course where students build functionality incrementally.

### Pinned LangChain4j Version

Pinned to **LangChain4j 1.15.0**. See the [release notes](https://github.com/langchain4j/langchain4j/releases) for the per-version history. Recent additions exercised in this course: agentic API (1.8 → 1.14) plus the voting pattern in `langchain4j-agentic-patterns` (1.15), MCP spec 2025-11-25, OpenAI transcription model, `gpt-image-2` image generation, `ChatMemory.set()`, `Optional` and `@P(defaultValue = ...)` tool parameters, per-call `ChatRequestParameters`, streaming cancellation, hybrid search in PgVector / Elasticsearch.

### Repository Structure

- **`main` branch**: Starter code with TODO-guided exercises for students
- **`solutions` branch**: Complete working implementations for reference
- **Test classes**: Contain TODO comments guiding students through implementation
- **Example classes**: Skeleton implementations with TODO instructions

The course demonstrates integration of Large Language Models (LLMs) with Java applications using the LangChain4j library (1.15.0), covering:

- Text generation and chat capabilities
- Structured data extraction
- Prompt engineering with templates
- Chat memory (single-user, multi-user, and `set()`-based replacement)
- Function calling with local @Tool methods (incl. `Optional` parameters and `@P(defaultValue = ...)`)
- External tool integration via Model Context Protocol (MCP) — spec 2025-11-25
- Vision capabilities for image understanding and `gpt-image-2` generation
- Audio transcription via OpenAI's dedicated transcription model
- Retrieval-Augmented Generation (RAG) with PDF and web content
- Agentic API: composing multi-step LLM workflows (sequence, loop, conditional, parallel, voting)

## Common Commands

### Build and Run

```bash
# Build the project
./gradlew build

# Run example main classes directly (no application runner needed)
./gradlew run --main-class=com.kousenit.langchain4j.examples.BasicChatExample

# Run tests with environment variables
OPENAI_API_KEY=your_key ./gradlew test
```

### Code Formatting

This project uses [Spotless](https://github.com/diffplug/spotless) with [Palantir Java Format](https://github.com/palantir/palantir-java-format) for consistent code style:

```bash
# Check if code is properly formatted
./gradlew spotlessCheck

# Apply formatting to all code
./gradlew spotlessApply
```

The formatting is automatically configured in `build.gradle.kts` and uses Palantir Java Format 2.90.0, which is compatible with Java 25.

### Slides PDF Publishing

The published PDF is **not committed to the repo**. `.github/workflows/build-slides-pdf.yml` auto-builds on pushes to `main` or `solutions` that touch `slides/slides.md`, `package.json`, `pnpm-lock.yaml`, or the workflow itself, then attaches `slides-export.pdf` to a rolling `slides-latest` release. Stable URL:

```text
https://github.com/kousen/LangChain4j_Training_course/releases/latest/download/slides-export.pdf
```

Local commands:

```bash
pnpm install
pnpm export
```

### Testing

```bash
# Run all tests (many will be empty TODO stubs in main branch)
./gradlew test

# Run specific test classes (students implement these progressively)
./gradlew test --tests OpenAiChatTests
./gradlew test --tests AnthropicChatTests
./gradlew test --tests RAGTests

# Run individual test methods
./gradlew test --tests OpenAiChatTests.basicChat

# To see working tests, switch to solutions branch
git checkout solutions
./gradlew test
```

### Chroma Setup (for Lab 10 vector storage)

```bash
# Start Chroma for the vector-store lab
docker run -p 8000:8000 chromadb/chroma:0.5.4
```

### Issue Management

```bash
# Create a new GitHub issue
gh issue create --title "Issue Title" --body "Issue description"

# List open issues
gh issue list

# Close an issue
gh issue close <issue-number>
```

**Important**: Always create GitHub issues for new features, major refactors, or bug fixes before starting work. This helps with project tracking and documentation.

**CRITICAL REMINDER**: Before implementing any significant changes or new features:
1. **CREATE** a GitHub issue first using `gh issue create`
2. **IMPLEMENT** the feature or fix
3. **CLOSE** the issue when complete using `gh issue close <number>`

This workflow ensures proper documentation and project tracking. Don't forget to close issues upon completion!

## Lab Implementation Workflow

Follow this established workflow when implementing each lab:

### 1. Create GitHub Issue
```bash
gh issue create --title "Lab X: [Lab Title]" --body "Implement Lab X exercises following the labs.md specification"
```

### 2. Create Feature Branch (from main)
```bash
git checkout main
git pull origin main
git checkout -b labX-[descriptive-name]
```

### 3. Implement Starter Code (on feature branch)
- Create test class with TODO comments guiding students
- Include proper imports and basic structure

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kousen/LangChain4j_Training_course](https://github.com/kousen/LangChain4j_Training_course) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
