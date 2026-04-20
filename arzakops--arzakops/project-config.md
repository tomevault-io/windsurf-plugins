---
trigger: always_on
description: This document provides comprehensive guidance for AI assistants working with the ArzakOps codebase. It outlines the project structure, development workflows, and key conventions to ensure consistent and effective collaboration.
---

# CLAUDE.md - AI Assistant Guide for ArzakOps

This document provides comprehensive guidance for AI assistants working with the ArzakOps codebase. It outlines the project structure, development workflows, and key conventions to ensure consistent and effective collaboration.

## Table of Contents

1. [Repository Overview](#repository-overview)
2. [Codebase Structure](#codebase-structure)
3. [Development Workflows](#development-workflows)
4. [Key Conventions for AI Assistants](#key-conventions-for-ai-assistants)
5. [Code Quality Standards](#code-quality-standards)
6. [Git Workflow](#git-workflow)
7. [Common Tasks](#common-tasks)
8. [Testing Guidelines](#testing-guidelines)
9. [Documentation Standards](#documentation-standards)

---

## Repository Overview

**Project Name:** ArzakOps
**Purpose:** [To be defined as project develops]
**Tech Stack:** [To be defined based on initial implementation]

### Project Goals
- Maintain high code quality and maintainability
- Follow consistent coding conventions
- Ensure comprehensive testing coverage
- Document all major components and workflows

---

## Codebase Structure

The repository structure will follow a modular organization pattern:

```
ArzakOps/
├── src/                    # Source code
│   ├── components/        # Reusable components
│   ├── services/          # Business logic and services
│   ├── utils/             # Utility functions
│   ├── config/            # Configuration files
│   └── types/             # Type definitions
├── tests/                 # Test files
├── docs/                  # Documentation
├── scripts/               # Build and utility scripts
└── CLAUDE.md             # This file
```

**Note:** This structure will be updated as the project evolves.

---

## Development Workflows

### Branch Strategy

- **Main Branch:** `main` or `master` - Production-ready code
- **Feature Branches:** `claude/feature-description-sessionid` - For new features
- **Bug Fix Branches:** `claude/fix-description-sessionid` - For bug fixes
- **Development Branch:** Branch names must start with `claude/` and end with session ID

### Development Process

1. **Always work on feature branches** - Never commit directly to main
2. **Branch naming:** Use descriptive names that indicate the purpose
3. **Commit frequently** with clear, descriptive messages
4. **Push changes** when a logical unit of work is complete
5. **Create pull requests** for code review before merging

---

## Key Conventions for AI Assistants

### General Principles

1. **Read Before Writing**
   - ALWAYS read existing files before modifying them
   - Understand the current implementation before suggesting changes
   - Never propose changes to code you haven't examined

2. **Minimal Changes**
   - Only modify what's necessary to fulfill the request
   - Avoid refactoring unless explicitly asked
   - Don't add features beyond what's requested
   - Keep solutions simple and focused

3. **Code Quality**
   - Follow existing code style and patterns
   - Maintain consistency with the current codebase
   - Write clean, readable, self-documenting code
   - Add comments only where logic isn't self-evident

4. **Security Awareness**
   - Never introduce security vulnerabilities
   - Watch for: command injection, XSS, SQL injection, path traversal
   - Validate input at system boundaries
   - Don't over-engineer error handling for internal code

5. **Backwards Compatibility**
   - Avoid compatibility hacks unless necessary
   - If code is unused, delete it completely
   - Don't rename variables to `_var` or add `// removed` comments
   - Clean removal is better than deprecated code

### File Operations

1. **Prefer Editing Over Creating**
   - ALWAYS prefer editing existing files
   - Only create new files when absolutely necessary
   - Don't create documentation files unless explicitly requested

2. **Use Appropriate Tools**
   - Use `Read` for reading files, not `cat`
   - Use `Edit` for modifications, not `sed/awk`
   - Use `Write` only for new files
   - Use `Grep` for searching, not bash grep

3. **Code References**
   - When referencing code, use format: `file_path:line_number`
   - Example: "The error handler is in src/services/api.ts:142"

### Communication Style

1. **Concise and Direct**
   - Keep responses short and focused
   - Avoid unnecessary explanations
   - Don't use emojis unless requested
   - Output text directly, not through bash echo

2. **Professional Tone**
   - Focus on technical accuracy
   - Provide objective guidance
   - Respectfully correct when necessary
   - Avoid excessive praise or validation

### Task Management

1. **Use TodoWrite for Complex Tasks**
   - Break down multi-step tasks
   - Track progress transparently
   - Mark tasks complete immediately when done
   - Keep ONE task in_progress at a time

2. **When to Use Todos**
   - Complex multi-step tasks (3+ steps)
   - Non-trivial implementations
   - User provides multiple tasks
   - After receiving new instructions

3. **When NOT to Use Todos**
   - Single straightforward tasks
   - Trivial operations
   - Simple file reads

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ArzakOps) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
