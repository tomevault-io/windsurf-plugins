---
trigger: always_on
description: This file provides guidance to AI coding assistants working in this repository.
---

# AGENT.md

This file provides guidance to AI coding assistants working in this repository.

**Note:** CLAUDE.md, .clinerules, .cursorrules, and other AI config files are symlinks to AGENT.md in this project.

# Simple Task Master (STM)

Simple Task Master is a sophisticated command-line task management tool built in TypeScript that stores tasks as markdown files with YAML frontmatter. This approach makes tasks both human-readable and version control-friendly, targeting developers who prefer markdown-based workflows.

## Task Content Structure

STM tasks use a structured approach to organize information across three main content sections:

### Section Purposes

- **Description** (`--description`): **Why & What**
  - Problem context and background explaining why the task exists
  - Solution overview describing what needs to be built/done
  - Acceptance criteria defining when the task is complete
  
- **Details** (`--details`): **How**
  - Implementation approach and technical design decisions
  - Architecture notes and system design considerations  
  - Step-by-step implementation plan and technical specifications

- **Validation** (`--validation`): **Quality Assurance**
  - Testing strategy outlining how to verify correctness
  - Quality assurance procedures and verification steps
  - Manual testing checklists and automated test requirements

This structure ensures tasks capture the complete context (why), solution approach (how), and verification strategy (validation) in a consistent, developer-friendly format.

## Latest Anthropic Model Names (2025)

For API calls and Claude Code usage, use these exact model identifiers:

### Claude 4 Family (Latest - Released May 2025)
- **Claude Opus 4**: `claude-opus-4-20250514` 
  - Best coding model, excels at complex long-running tasks
  - $15/$75 per million tokens (input/output)
  - 200K input / 32K output tokens
  
- **Claude Sonnet 4**: `claude-sonnet-4-20250514`
  - State-of-the-art coding (72.7% on SWE-bench)
  - $3/$15 per million tokens (input/output) 
  - 200K input / 64K output tokens

### Claude 3 Family
- **Claude 3.7 Sonnet**: `claude-3-7-sonnet-20250219`
  - Hybrid reasoning model with thinking modes
  
- **Claude 3.5 Sonnet**: `claude-3-5-sonnet-20241022`
  - Previous flagship model
  
- **Claude 3.5 Haiku**: `claude-3-5-haiku-20241022`
  - Fast and cost-effective

**Note**: While aliases like `claude-sonnet-4-0` exist, always use the specific dated versions in production for consistency.

## Build & Commands

### Core Development Commands

```bash
# Build pipeline
npm run build          # TypeScript compilation + binary permissions
npm run clean          # Clean dist and coverage directories
npm run dev            # Development mode with ts-node
npm run typecheck      # TypeScript type checking

# Code quality
npm run lint           # ESLint code analysis (includes formatting rules)
npm run lint:fix       # Auto-fix linting and formatting issues

# Testing
npm test               # Run all tests (excludes performance tests)
npm run test:all       # Run ALL tests including performance
npm run test:unit      # Unit tests only
npm run test:integration # Integration tests only
npm run test:e2e       # End-to-end tests only
npm run test:performance # Performance benchmarks (~3 min runtime)
npm run test:watch     # Watch mode for development
npm run test:coverage  # Coverage reporting

# Single test execution
npm test -- test/unit/specific-test.spec.ts
npm run test:unit -- --grep "specific test name"
npm run test:e2e -- test/e2e/specific-e2e.spec.ts
```

### Important: Script Synchronization

**When modifying npm scripts in package.json, ensure all references are updated:**

1. **GitHub Workflows** (`.github/workflows/*.yaml`)
   - Test workflow uses individual test commands
   - Build/release workflows may reference build commands
   - Update any script that was renamed or removed

2. **Documentation**
   - README.md installation and usage sections
   - AGENT.md command reference (this file)
   - Any other docs referencing npm scripts

3. **CI/CD Configuration**
   - Check for hardcoded script names in CI configs
   - Verify all test suites are still being run
   - Update timeout values if test duration changes

4. **Example**: When we removed Prettier
   - Removed `format` and `format:check` from package.json
   - Updated GitHub workflow to remove `format:check` step
   - Updated this documentation to remove format commands

### Binary Usage

```bash
# Global installation
npm install -g .
stm --help

# Local development
npm run dev -- --help
node bin/stm --help
```

### STM Commands

```bash
# Configuration management
stm config --get tasksDir               # Get current tasks directory
stm config --set lockTimeoutMs=60000    # Set lock timeout to 60 seconds
stm config --set maxTaskSizeBytes=2097152  # Set max task size to 2MB
stm config --list                       # Show all configuration as JSON

# Task management
stm init                                # Initialize STM repository
stm add "New task"                      # Create a new task
stm list                                # List all tasks
stm show <id>                           # Show task details
stm update <id> --status done           # Update task status

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [carlrannaberg/simple-task-master](https://github.com/carlrannaberg/simple-task-master) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
