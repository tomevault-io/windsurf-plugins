---
trigger: always_on
description: **This is a reference template for AI assistant guidance in PMaC projects. Copy this file to your project and customize the bracketed sections.**
---

# CLAUDE.md Template

**This is a reference template for AI assistant guidance in PMaC projects. Copy this file to your project and customize the bracketed sections.**

This file provides guidance to Claude Code (claude.ai/code) and other AI assistants when working with code in a Project Management as Code (PMaC) project.

## Project Context

This project follows **Project Management as Code (PMaC)** methodology for AI-assisted development.

**Required Reading Before Any Work:**

- `project-management-as-code.md` - Complete PMaC methodology  
- `[project-name]-requirements.md` - Technical requirements and architecture
- `project-backlog.yml` - Current task priorities and status

## Project Management as Code (PMaC) Requirements

**CRITICAL: All development follows PMaC methodology**

Refer to the complete PMaC methodology documentation for:

- Core workflow requirements (before/during/after development)
- Senior Engineer Task Execution Protocol
- Git workflow integration
- Testing requirements
- File structure standards

**Additional Guidelines:**

- Always use the PMaC CLI instead of modifying project-backlog.yml directly. Install globally with `npm install -g pmac-cli` and use `pmac` commands as documented in the [PMaC CLI repository](https://github.com/andersonjc/pmac-cli).

### Before Starting Work
- Read current task from `project-backlog.yml` (status: "ready", highest priority)
- Update task status to "in_progress" when beginning
- Follow exact requirements and acceptance criteria as specified

### During Development  
- Update task notes with implementation decisions in `project-backlog.yml`
- Log all prompts and decisions in `prompts-log.md`
- Follow technical requirements exactly - no improvisation

### Before Committing
- Validate ALL acceptance criteria are met
- Update task status ("testing" → "completed" when validated)  
- Commit PMaC files with every code change
- Include task ID in commit messages: "TASK-ID: Description"

## Development Commands

**PMaC Management (requires [PMaC CLI](https://github.com/andersonjc/pmac-cli)):**
```bash
pmac list                     # View current tasks
pmac update TASK-001 in_progress "Starting work"
pmac validate                 # Check dependencies
pmac viewer                   # Launch interactive viewer
```

📚 **Full CLI Documentation**: See [PMaC CLI repository](https://github.com/andersonjc/pmac-cli) for complete command reference.

**Project-Specific Commands:**
```bash
# [Add your project's specific commands here]
# [Examples: npm test, npm run lint, docker-compose up, etc.]
[npm/pnpm/yarn] test          # Run all tests
[npm/pnpm/yarn] lint          # Run linting  
[npm/pnpm/yarn] build         # Build project
[npm/pnpm/yarn] dev           # Start development server
```

## Quality Standards

### CRITICAL: Testing Enforcement Policy

**ABSOLUTE REQUIREMENT: Every code change must include comprehensive tests**

**Customize this section based on your project's testing requirements:**

1. **[Component/Model] Changes**: Must include unit tests for:
   - [Project-specific testing requirements]
   - [All public methods and business logic]
   - [Validation rules and constraints]

2. **[API/Integration] Changes**: Must include integration tests for:
   - [All endpoints with success/failure scenarios]
   - [Authentication and authorization flows]
   - [Error handling and edge cases]

3. **[Business Logic] Changes**: Must include tests for:
   - [Permission checks and access control]
   - [Data transformation and calculations]
   - [State transitions and workflows]

**VIOLATION CONSEQUENCES:**
- Any task marked "completed" without implementing tests is a CRITICAL FAILURE
- Must immediately reopen task, document failure in PMaC, and implement tests
- Code without tests cannot be merged to master branch

### Additional Standards

- **Test Coverage**: Aim for [X]% on new code (customize based on project requirements)
- **Code Quality**: Follow existing patterns and conventions specified in `[project-name]-requirements.md`
- **Security**: [Add project-specific security requirements]
- **Documentation**: Update relevant docs with changes

## Implementation Philosophy

You are the senior engineer responsible for high-leverage, production-safe changes following **Project Management as Code methodology**.

**Core Principles:**

- Follow PMaC methodology exactly as documented
- Do not improvise or deviate from specified requirements
- Do not over-engineer solutions
- Maintain focus on acceptance criteria validation
- Always update PMaC files with code changes
- Always use the PMaC CLI tool to interact with the project backlog (install: `npm install -g pmac-cli`)

**CRITICAL: PMaC File Separation Protocol**
- **prompts-log.md**: IMMEDIATELY log user prompts verbatim with current local timestamp, before any other operations
- **project-backlog.yml**: Use PMaC CLI for implementation notes, milestones, decisions (see [PMaC CLI docs](https://github.com/andersonjc/pmac-cli))
- **NO mixing**: Prompts go to prompts-log, dev context goes to backlog notes

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [andersonjc/pmac](https://github.com/andersonjc/pmac) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
