---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a **hands-on training course** for learning Spring AI through progressive lab exercises. The repository is structured as a proper training course where students build functionality incrementally.

### Repository Structure

- **`main` branch**: Starter code with TODO-guided exercises for students
- **`solutions` branch**: Complete working implementations for reference
- **`labs.md`**: 15 progressive lab exercises with step-by-step instructions
- **`slides.md`**: Comprehensive Slidev presentation for training sessions
- **Test classes**: Contain TODO comments guiding students through implementation
- **Service classes**: Skeleton implementations with TODO instructions

The course demonstrates integration of Large Language Models (LLMs) with Spring applications using the Spring AI library (version 1.1.4), covering:

- Text generation and chat capabilities
- Structured data extraction  
- Prompt engineering with templates
- Chat memory for maintaining conversation context
- Vision capabilities for image understanding and generation
- Audio processing (text-to-speech and speech-to-text)
- Retrieval-Augmented Generation (RAG) with PDF and web content
- Model Context Protocol (MCP) for standardized tool integration

## Common Commands

### Build and Run

```bash
# Build the project
./gradlew build

# Run the application (default profile)
./gradlew bootRun

# Run with RAG profile enabled
./gradlew bootRun --args='--spring.profiles.active=rag'

# Run with both RAG and Redis profiles
./gradlew bootRun --args='--spring.profiles.active=rag,redis'

# Run with MCP client functionality
./gradlew bootRun --args='--spring.profiles.active=mcp'

# Run with MCP server functionality
./gradlew bootRun --args='--spring.profiles.active=mcp-server'
```

### Testing

```bash
# Run all tests (many will be empty TODO stubs in main branch)
./gradlew test

# Run specific test classes (students implement these progressively)
./gradlew test --tests OpenAiTests
./gradlew test --tests ClaudeTests
./gradlew test --tests RAGTests

# Run with specific profiles (for advanced RAG exercises)
./gradlew test --tests RAGTests -Dspring.profiles.active=rag,redis

# Run MCP tests (note: may fail when run together due to profile conflicts)
./gradlew test --tests McpServerTests
./gradlew test --tests McpClientTests

# To see working tests, switch to solutions branch
git checkout solutions
./gradlew test
```

### Redis Setup (for RAG with Redis vector store)

```bash
# Start Redis Stack container
docker run -p 6379:6379 redis/redis-stack:latest
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

## CRITICAL: Branch Management Guidelines

**⚠️ NEVER merge main branch into solutions branch without careful review!**

### Branch Purposes
- **`main` branch**: Starter code with TODO stubs for students
- **`solutions` branch**: Complete working implementations for reference

### Safe Merge Practices
1. **Before any merge**: Always check target branch has complete implementations
2. **Use selective merging**: Cherry-pick specific commits rather than full merges
3. **Documentation-only merges**: Only merge documentation/config changes, never test code
4. **Verify after merge**: Run tests to ensure solutions still work

### Emergency Recovery
If solutions are accidentally overwritten:
```bash
# Find the last good commit with complete implementations
git log --oneline solutions --grep="complete\|implement\|working"

# Restore specific files from earlier commit
git checkout <good-commit-hash> -- src/test/java/com/oreilly/springaicourse/
git checkout <good-commit-hash> -- src/main/java/com/oreilly/springaicourse/

# Commit the restoration
git commit -m "Restore complete implementations from backup"
```

**Remember**: Solutions branch should NEVER have TODO comments in test methods!

## Required Environment Variables

Set these environment variables before running the application:

```bash
export OPENAI_API_KEY=your_openai_api_key
export ANTHROPIC_API_KEY=your_anthropic_api_key  # Optional, for Claude exercises
```

## Common Tasks

### Adding Navigation to Exercise Files

To add a table of contents with navigable links to any tutorial/exercise file:

1. Add a table of contents section at the top like this:
```markdown
## Table of Contents

- [Exercise 1: Basic Setup](#exercise-1-basic-setup)
- [Exercise 2: Advanced Features](#exercise-2-advanced-features)
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kousen/Spring_AI_Training_Course](https://github.com/kousen/Spring_AI_Training_Course) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
