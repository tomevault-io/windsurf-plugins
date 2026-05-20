---
trigger: always_on
description: > This file follows the AGENTS.md standard, supported by Cursor, GitHub Copilot, Codex, Aider, and other AI coding tools.
---

# AGENTS.md - Universal AI Agent Configuration

> This file follows the AGENTS.md standard, supported by Cursor, GitHub Copilot, Codex, Aider, and other AI coding tools.
> Place this file in your project root and customize the sections below for your project.

# [Project Name]

## Project Overview
[Brief description of the project's purpose and core functionality]

**Architecture**: [e.g., Microservices, Monolith, JAMstack]
**Domain**: [e.g., E-commerce, Healthcare, Finance]
**Scale**: [e.g., Startup MVP, High-traffic production, Enterprise]

## Tech Stack

### Core Technologies
- **Language**: [Python 3.11+, TypeScript 5.x, etc.]
- **Framework**: [FastAPI, Next.js, Spring Boot, etc.]
- **Database**: [PostgreSQL 15, MongoDB 6, etc.]
- **Cache**: [Redis 7, Memcached, etc.]
- **Package Manager**: [npm, pnpm, pip, poetry, etc.]

### Key Dependencies
- [Library/Framework]: [Version and purpose]
- [Library/Framework]: [Version and purpose]

### Development Tools
- **Linter**: [ESLint, Pylint, etc.]
- **Formatter**: [Prettier, Black, etc.]
- **Type Checker**: [TypeScript, mypy, etc.]
- **Testing**: [Vitest, pytest, JUnit, etc.]

## Setup Commands

```bash
# Install dependencies
[install command]

# Setup environment
cp .env.example .env
# Edit .env with required values

# Database setup
[migration commands]

# Start development server
[dev command]
```

## Build and Development Commands

### File-Scoped Commands (Preferred - Fast)
```bash
# Type check single file (3 seconds)
[command] path/to/file.ext

# Format single file
[command] path/to/file.ext

# Lint single file
[command] path/to/file.ext

# Run single test file
[command] path/to/test.ext
```

### Project-Wide Commands (Use Sparingly)
```bash
# Full type check (2 minutes)
[command]

# Full test suite (3 minutes)
[command]

# Full build (5 minutes)
[command]
```

**Important**: Always use file-scoped commands for checking individual changes. Only run project-wide commands when explicitly requested or before final commit.

## Code Style and Conventions

### Language-Specific Rules
- [e.g., Use TypeScript strict mode with exactOptionalPropertyTypes]
- [e.g., Python type hints required for all functions]
- [e.g., Prefer composition over inheritance]

### Naming Conventions
- **Files**: [e.g., lowercase-with-dashes for directories, PascalCase for components]
- **Variables**: [e.g., camelCase with auxiliary verbs: isLoading, hasError]
- **Functions**: [e.g., camelCase, descriptive action verbs]
- **Classes**: [e.g., PascalCase, noun-based names]
- **Constants**: [e.g., UPPER_SNAKE_CASE]

### Formatting Rules
- **Indentation**: [tabs/spaces and size]
- **Line Length**: [max characters]
- **Quotes**: [single/double]
- **Semicolons**: [required/optional]
- **Trailing Commas**: [yes/no]
- **Comments**: [Use JSDoc docstrings / follow PEP 257 / etc.]

### Do's ✅
- [Specific pattern to use - reference actual file]
- [Another approved pattern - reference actual file]
- [Framework-specific best practice]
- [Design pattern to follow]

### Don'ts ❌
- [Specific anti-pattern to avoid - reference bad example if exists]
- [Another pattern to avoid]
- [Deprecated approach]
- [Common mistake specific to your project]

### Good and Bad Examples

**Avoid** (Anti-patterns):
- [File/pattern to avoid]: [Brief explanation why]

**Prefer** (Good patterns):
- [File/pattern to copy]: [Brief explanation why]
- [For feature X, copy]: `path/to/good/example.ext`

## Project Structure

```
/
├── [directory]/          # [Description and what lives here]
├── [directory]/          # [Description]
│   ├── [subdirectory]/   # [Description]
│   └── [subdirectory]/   # [Description]
└── [directory]/          # [Description]
```

**Key Files**:
- `[path/to/important/file]`: [What it does]
- `[path/to/important/file]`: [What it does]

**Architecture Patterns**:
- [Pattern used, e.g., Repository pattern for data access]
- [Pattern used, e.g., Service layer for business logic]
- [State management approach]

## Testing Instructions

### Testing Framework
- **Unit Tests**: [Framework and location]
- **Integration Tests**: [Framework and location]
- **E2E Tests**: [Framework and location]

### Running Tests
```bash
# Run all tests
[command]

# Run specific test file
[command] path/to/test.ext

# Run tests with coverage
[command]

# Run tests in watch mode
[command]
```

### Test Requirements
- **Coverage Target**: [e.g., 85% minimum]
- **Test Location**: [e.g., `*.test.ts` alongside source, `tests/` directory]
- **Test Naming**: [e.g., Omit "should" from test names]
- **Mocking Strategy**: [e.g., Mock external APIs, use test database]

### Testing Patterns
```[language]
# Example test structure
[Paste a good test example from your codebase]
```

## Security Considerations

### Authentication & Authorization
- [e.g., JWT-based authentication]
- [e.g., Role-based access control implemented via middleware]
- [e.g., All non-public endpoints require authentication]

### Input Validation
- [e.g., Validate all user inputs on both client and server]
- [e.g., Use Zod schemas for validation]
- [e.g., Sanitize inputs to prevent XSS]

### Secrets Management
- **NEVER** commit secrets, API keys, or credentials

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [obviousworks/vibe-coding-ai-rules](https://github.com/obviousworks/vibe-coding-ai-rules) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
