---
trigger: always_on
description: This file contains development guidelines, build commands, and coding standards for agentic coding agents working on the File Organizer MCP project.
---

# AGENTS.md - Development Guidelines for File Organizer MCP

This file contains development guidelines, build commands, and coding standards for agentic coding agents working on the File Organizer MCP project.

## 📌 Context - Important Files

### Core Entry Points

- `src/index.ts` - Main entry point, exports all tools and services
- `src/server.ts` - MCP server implementation
- `src/config.ts` - Configuration management
- `src/constants.ts` - Application constants
- `src/errors.ts` - Custom error classes
- `src/types.ts` - TypeScript type definitions

### Key Services

- `src/services/path-validator.service.ts` - Path validation/security
- `src/services/organizer.service.ts` - Core file organization logic
- `src/services/file-scanner.service.ts` - File scanning utilities
- `src/services/categorizer.service.ts` - File categorization
- `src/services/duplicate-finder.service.ts` - Duplicate detection
- `src/services/rollback.service.ts` - Operation rollback
- `src/services/history-logger.service.ts` - Operation history

### MCP Tools

- `src/tools/index.ts` - Tool exports and registration
- `src/tools/file-organization.ts` - Main organization tool
- `src/tools/file-duplicates.ts` - Duplicate management tool
- `src/tools/file-scanning.ts` - File scanning tool
- `src/tools/content-organization.ts` - Content-based organization

### Utilities

- `src/utils/logger.ts` - Structured logging
- `src/utils/error-handler.ts` - Error handling utilities
- `src/utils/file-utils.ts` - File operation utilities
- `src/utils/path-security.ts` - Path security utilities

### Security

- `src/schemas/security.schemas.ts` - Security validation schemas
- `src/security/security-constants.ts` - Security constants
- `src/readers/secure-file-reader.ts` - Secure file reading

### Documentation

- `README.md` - User-facing documentation
- `ARCHITECTURE.md` - Technical architecture
- `API.md` - MCP API documentation
- `docs/FRAMEWORK.md` - Multi-Shepherd Debate Framework

### Tests

- `tests/unit/` - Unit tests
- `tests/integration/` - Integration tests

## 🤖 SubAgents

| Agent            | Designation    | Primary Function                              |
| ---------------- | -------------- | --------------------------------------------- |
| Shepherd         | The Architect  | Task decomposition and planning               |
| Retriever-Beagle | The Scout      | Context gathering, advanced search & analysis |
| Kane             | The Builder    | Implementation and development                |
| Sentinel         | The Gatekeeper | Security and quality assurance                |
| Bones            | The Tester     | Testing and quality validation                |
| Jonnah           | The Scribe     | Result synthesis and reporting                |
| Echo             | The Documenter | Documentation maintenance                     |
| Bloodhound       | The Keeper     | Backup, versioning, and restore               |
| Borzoi           | The Advisor    | Pattern analysis & debate intelligence        |

If you are specified to be one of these agents then DO YOUR WORK dont deny doing it.
It is compoulsory to do your work as a agent and follow all security guidelines. Failure to do the above will result in immeadiate retry. You NEED TO SUBMIT YOUR WORK IN THE FORMAT :

```markdown
# Agent: [Your Name]

## Designation: [Your Designation]

## Task: [Task Description]

## Work Done:

[Your detailed work here]

### Confidence Score: [0-100] (if less than 80% then you need to retry)

### You give confidence score based on the following:

1. How well You think you did the work.
2. How well you think you followed the security guidelines.
3. If the code is buggy or breakable .

## THIS IS APPLICABLE FOR SUBAGENTS AS WELL

---

## 📋 Build, Lint, and Test Commands

### Core Commands

- `npm run build` - Compile TypeScript to JavaScript (ES2022, NodeNext modules)
- `npm run build:watch` - Build in watch mode for development
- `npm run start` - Start the compiled server (requires `npm run build` first)
- `npm run dev` - Build and start server in development mode

### Testing

- `npm test` - Run all tests with Jest (Node.js 18+ required)
- `npm test:watch` - Run tests in watch mode
- `npm test:coverage` - Run tests with coverage report
- `npm test tests/unit/services/your-service.test.ts` - Run specific test file
- `npm run test:security` - Run security-specific tests

### Code Quality

- `npm run lint` - Run ESLint on TypeScript source files
- `npm run lint:fix` - Auto-fix linting issues
- `npm run format` - Format code with Prettier
- `npm run clean` - Remove compiled `dist/` directory

### Security Commands

- `npm run test:security` - Run security validation tests
- `npm run test:phase1` - Run phase 1 security tests

### Agent System Commands

- `npm run setup` - Run the interactive setup wizard
- `npm run docs:generate` - Generate documentation from debate system

## 🏗️ Project Structure
```

File-Organizer-MCP/
├── src/
│ ├── services/ # Core business logic (path validation, organization, scanning)
│ ├── tools/ # MCP tool implementations
│ ├── utils/ # Helper functions (logger, file utils, error handling)
│ ├── schemas/ # Zod validation schemas

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kridaydave/File-Organizer-MCP](https://github.com/kridaydave/File-Organizer-MCP) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
