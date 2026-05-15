---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Eye-Rolling Workflow is a specification-driven development workflow command set designed for Claude Code and other AI assistants. The name is self-deprecating — from an outsider's perspective, the back-and-forth clarification process with AI can seem annoyingly thorough. But that's the point: think it through before you code.

The project provides a structured three-step methodology: from requirement clarification to complete implementation, ensuring the development process is clear and traceable through PRD (Product Requirements Document) driven development.

The core value lies in ensuring every feature goes through thorough requirement analysis, has clear implementation plans, and maintains a trackable, interruptible, and resumable execution process.

## Core Architecture

The project follows a three-step specification-driven workflow:

### Step 1: Requirement Clarification (`/create-prd`)
- Interactive Q&A to clarify feature requirements
- Generate comprehensive Product Requirements Document (PRD)
- Output: `docs/specs/[date]-[feature-name]/prd.md`
- Target audience: Product managers and business people
- Focus: Business value, user needs, high-level technical direction

### Step 2: Implementation Planning (`/create-impl-plan`)
- Analyze PRD and existing codebase architecture
- Generate 5-7 specific implementation tasks with detailed implementation points
- Generate "Completion Checks" for each task with executable verification methods
- Automatic Gherkin acceptance test generation
- Output: `implementation.md` + `acceptance.feature`
- Target audience: Developers (junior to intermediate)
- **Enhanced Feature**: Implementation reference information extraction from research and PRD files

### Step 3: Task Execution (`/process-task-list`)
- Execute implementation tasks one by one with automatic git commits
- Execute "Completion Checks" after each task to verify implementation
- Acceptance testing integration with specialized acceptance-tester skill
- Support for multi-session execution for long-term development
- Reference documents: implementation.md, PRD, acceptance.feature, research (if available)

### Workflow Context Awareness

Each command includes a "Workflow Context" section that provides:
- **Global View**: Understanding of the complete three-step workflow
- **Stage Position**: Current stage in the overall development process
- **Input/Output Clarity**: What information comes in and what should be produced
- **Responsibility Boundaries**: What should and should not be included in outputs
- **Stage Relationships**: How current stage connects with previous and next stages

This design ensures:
- **Separation of Concerns**: Each stage focuses on its specific responsibilities
- **Content Distribution**: Technical details flow from research → PRD (reference only) → implementation.md (detailed extraction)
- **Audience Alignment**: PRD for business stakeholders, implementation.md for developers
- **Context Preservation**: Technical insights preserved through "Implementation Reference" sections

### Supporting Components

#### Installation System (`/scripts/`)
- **install-config.js**: Automated installation to `~/.claude/commands`
- Cross-platform compatibility and conflict detection

#### Documentation Output (`/docs/`)
- **research/**: Research documents (optional auxiliary feature)
- **specs/**: Complete specification folders with PRD, implementation plans, and acceptance tests

## Common Commands

### Development Setup
```bash
# Install workflow commands to Claude Code
npm run install-config

# Force overwrite existing commands
npm run install-config -- overwrite
```

### Project Dependencies
- **shelljs**: For cross-platform shell operations in installation scripts
- **Node.js**: Required for installation scripts execution

## Key Workflow Patterns

### 1. Research-Driven Development
- Start with `/research` command for complex problems
- Output: `docs/research/[date]-[topic].md`
- Focus on problem understanding before solution design

### 2. PRD-First Feature Development
- Use `/create-prd` for new feature requirements
- Output: `docs/specs/[date]-[feature-name]/prd.md`
- Target audience: Product managers and business people
- Focus on business value and user needs, reference (not copy) technical details from research

### 3. Implementation Planning
- Use `/create-impl-plan` to convert PRD to actionable tasks
- Output: Implementation markdown + Gherkin acceptance tests
- Integrates with Claude Code's TodoWrite tool
- Each task includes "Completion Checks" for verification

### 4. Acceptance Testing Integration
- Automatic detection of acceptance testing tasks
- Launches specialized acceptance-tester skill with complete file context
- Executes Gherkin scenarios through commands and browser automation
- Receives implementation.md, acceptance.feature, and prd.md for comprehensive testing

## File Naming Conventions

- Research documents: `[YYYY-MM-DD]-[topic-slug].md`
- Spec folders: `[YYYY-MM-DD]-[feature-name]/`
- Spec contents: `prd.md`, `implementation.md`, `acceptance.feature`

## Installation Locations


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yurenju/eye-rolling-workflow](https://github.com/yurenju/eye-rolling-workflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
