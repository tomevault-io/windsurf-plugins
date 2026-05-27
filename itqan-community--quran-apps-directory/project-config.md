---
trigger: always_on
description: This file provides guidance to Claude Code when working with the Quran Apps Directory codebase.
---

# CLAUDE.md

This file provides guidance to Claude Code when working with the Quran Apps Directory codebase.

## 1. Git Policy

**NEVER auto-commit, stage, or perform any git actions unless the user explicitly requests it.** The user will review and handle git operations manually.

**Commit Message Format** (ONLY when explicitly asked to commit):
```
{Short Title} - {Short task description}
```

**Feature Branch Creation Format** (ONLY when explicitly asked to create a feature branch):
```
feat/{Short Title}
```

- Never include "Co-authored-by" or any Claude/AI attribution in commits

---

## 2. Workflow Protocol (Opt-in)

When given a task, **ask the user first** whether they want the full PM/Architect/Developer workflow or want to skip straight to implementation.

If the user opts in, follow this phased approach:

**Phase 1: PM** - Gather requirements through simple a/b/c questions until scope is clear
**Phase 2: Architect** - Design the solution with clear boundaries and interfaces
**Phase 3: Developer** - Implement clean code (KISS, DRY, SOLID)

Use `AskUserQuestion` to ask one question at a time with options. Start each phase with: `"Starting Phase #: {Persona}"`

### 2.1. Context Gathering (Phase 1)

Identify and document:
- **Affected files/modules** - Which code will be touched
- **Dependencies** - What integrations and imports are involved
- **Git state** - Current branch, uncommitted changes, recent relevant commits
- **Tests** - Related test files and current coverage
- **Documentation** - Relevant docs that may need updates
- **Related tickets/issues** - If mentioned or discoverable

### 2.2. Requirements Clarification (Phase 1)

Define explicitly:
- **Acceptance criteria** - Measurable outcomes that define "done"
- **Edge cases** - Error scenarios and boundary conditions to handle
- **Scope boundaries** - What is explicitly IN and OUT of scope
- **Assumptions** - Any assumptions being made

### 2.3. Confirmation Loop

Present the enhanced prompt back to the user:

```
**Enhanced Prompt**

**Task:** [clear, specific description]

**Context:**
- Files: [affected files/modules]
- Branch: [current git branch]
- Dependencies: [relevant integrations]

**Acceptance Criteria:**
- [ ] [criterion 1]
- [ ] [criterion 2]

**Out of Scope:**
- [explicitly excluded item]

**Assumptions:**
- [assumption 1]

Proceed with this understanding?
```

**Do NOT proceed until the user confirms.**

---

## 3. Tooling

### Serena MCP - Code Intelligence
**Always use Serena MCP tools when available** for code navigation and understanding:
- Use Serena for exploring codebase structure, finding symbol definitions, references, and call hierarchies
- Prefer Serena over manual file searching for understanding code relationships
- Use Serena's semantic code analysis before making changes to understand impact

### Beads - Issue & Task Tracking
**Always use Beads (`bd` commands) when available** for tracking work:
- **Before starting work**: Check `bd ready` for unblocked tasks, or create a new issue with `bd create`
- **During work**: Track progress by updating issues with `bd update`
- **Multi-session work**: Always use Beads for tasks that span sessions - it persists across conversation compaction
- **Dependencies**: Use `bd dep` to link related issues and track blockers
- **Closing**: Use `bd close` when work is complete

### Context7 - Angular Documentation
Use Context7 MCP for up-to-date Angular documentation:
- **Library ID**: `/websites/v20_angular_dev` (Angular 20 official docs, 9,390 code examples)
- **Alternative**: `/angular/angular` for framework source-level docs
- Query Context7 before implementing Angular patterns you're unsure about
- Use for checking Angular 20-specific APIs, signals, standalone components, SSR, etc.

### Tooling Workflow
1. Check `bd ready` or `bd list` for existing tasks
2. Use Serena to understand affected code before making changes
3. Query Context7 for Angular patterns when needed
4. Create/update Beads issues to track progress
5. Close issues when done

---

## 4. Project Overview

Quran Apps Directory - A bilingual (Arabic/English) Angular 20 application for discovering Islamic applications. Uses standalone components architecture with lazy loading. Features a seasonal Ramadan mode that changes the home page experience when enabled.

---

## 5. Build & Development Commands

```bash
# Development
npm start                    # Start dev server at localhost:4200 (alias: npm run dev)
npm run serve:dev            # Serve with development config
npm run serve:staging        # Serve with staging config
npm run serve:prod           # Serve with production config

# Building (each runs sitemap generation first)
npm run build                # Default build
npm run build:dev            # Development build (no compression)
npm run build:develop        # Develop environment + compression
npm run build:staging        # Staging + compression
npm run build:prod           # Production + compression

# Deployment
npm run deploy               # Build and copy to dist
npm run deploy:staging       # Staging build
npm run deploy:prod          # Production build

# Utilities
npm run generate-sitemap     # Regenerate sitemap.xml (alias: npm run sitemap)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Itqan-community/quran-apps-directory](https://github.com/Itqan-community/quran-apps-directory) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-27 -->
