---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

### Environment Characteristics

- **The repo root is the pwd where Claude Code is opened. While ../../ is also the repo root due to using sprout, treat the pwd (worktree) as the repo root**

## Work Attitude and Quality Management

**Important**: Claude Code should act as a careful veteran engineer. Prioritize quality and reliability over implementation speed.

### Basic Principles
- Critically verify when you think implementation is complete
- Don't just think "it works", ask "does it really work correctly?"
- Actively consider potential problems and edge cases
- Don't skimp on testing and validation, be more thorough than necessary. Validation policy:
  - Actively use headless mode (calling the `HeadlessWish` class) for integrated verification except UI
  - Make actual communications to target machines and OpenAI API
  - Verify UI-related parts as much as possible yourself, but ask humans for difficult parts
- Always be skeptical of your own implementation and verify from multiple perspectives

### Pre-Implementation Checklist
- Detailed investigation of existing code (related files, dependencies, impact scope)
- Understanding of design patterns and coding conventions
- Consideration of test case coverage
- Verification of error handling appropriateness

### Post-Implementation Required Verification
1. **Code Quality**: Static analysis with `make lint`, `make format`
2. **Unit Tests**: Run all package tests with `make test`
3. **Integration Tests**: Comprehensive validation with `+uat`
4. **Manual Verification**: Confirm actual use case operation
5. **Error Cases**: Verify abnormal behavior
6. **Performance**: Test under unexpected load
7. **Compatibility**: Check impact on existing features

### Quality Standards
- All tests passing is the minimum requirement
- Zero static analysis errors is a prerequisite
- Confirmed to work in actual usage scenarios
- Appropriate behavior for edge cases and abnormal conditions
- Documentation updated (as needed)

### Error Handling and Fail-Fast Policy

**Important**: This repository adopts the fail-fast principle, prioritizing early error detection over recovery.

#### Basic Policy
- Immediately fail with exceptions for programming errors like type errors instead of handling with conditional branches
- Avoid continuing operation in uncertain states
- Emphasize early error detection and clear failures

#### Implementation Guidelines
- Immediately fail with exceptions for programming errors
- Only use WARN/ERROR logs for unexpected situations due to user environment factors
- Allow exceptional handling only for special reasons with explicit documentation

## Language Settings

Responses and document file generation should primarily be in English. Use English in the following cases:
- Comments, variable names, and function names in code
- Additions or modifications to existing English documentation
- When the user asks questions in English

## Specification Management

When developing new features, place specifications under `docs/(short-english-phrase-describing-feature)/*.md` in English.
These specifications serve as references during implementation, but more importantly, they help review "what was the purpose, how, and what was implemented" after implementation.

## Intermediate File Management

### Using the tmp/ Directory
All intermediate files created by Claude Code (test files, planning documents, temporary work files, etc.) must be created under the `tmp/` directory.

**Target Files**:
- Test sample files
- Implementation planning documents
- Debug temporary files
- Verification scripts
- Other temporary work files

**Exceptions**:
- Official project files (source code, configuration files, official documentation, etc.)
- When the user explicitly specifies another location

### Cleanup
The `tmp/` directory is regularly cleaned up:
- When executing the `+brc` command
- When executing the `+review` command
- During other quality management tasks

## Project Overview

wish is a Workflow-Aware AI Command Center that recognizes penetration testers' workflows and accelerates their thinking. By remembering and interpreting the results of commands executed by testers and suggesting the next logical move based on the situation, it dramatically reduces the cost of context switching.

## Key Development Commands

### Testing and Quality

```bash
# Basic commands (to be adjusted according to project structure)
make test             # Run unit tests
make lint            # Run ruff linting
make format          # Auto-format with ruff

# Python environment management
uv sync              # Install dependencies
uv sync --dev        # Install with dev dependencies
uv run pytest        # Run tests
```

### User Acceptance Testing (UAT)
```bash
# Complete system validation (shortcut command)
+uat                  # Run comprehensive UAT including code quality, tests, and functionality checks
```

### Running wish
```bash
# Installation
pip install wish-sh

# Basic usage
wish                  # Start the interactive CLI
```

## Architecture Overview

### Key Features

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AgenticSec/wish](https://github.com/AgenticSec/wish) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-28 -->
