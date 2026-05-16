---
trigger: always_on
description: This document contains guidelines for AI coding assistants. It is structured with generic best practices that apply to all projects, followed by project-specific sections that should be populated when starting a new project.
---

# AI Coding Assistant Guidelines

This document contains guidelines for AI coding assistants. It is structured with generic best practices that apply to all projects, followed by project-specific sections that should be populated when starting a new project.

## Generic Guidelines (All Projects)

### Project Management

#### Documentation Hierarchy

1. **CLAUDE.md is the source of truth** for all project specifications
2. README.md derives from CLAUDE.md with human-friendly summaries
3. In README.md, include: "For detailed specifications, see CLAUDE.md"
4. Update CLAUDE.md first, then sync README.md for major changes only
5. Keep CLAUDE.md in project root
6. **Use docs/ folder for topical documentation**:
   - Store technical docs, product requirements, and design specs
   - Use lowercase snake_case with date prefix (e.g., 20250517_color_system.md, 20250517_logging.md, 20250517_architecture.md)
   - **Required docs**: `docs/TODO.md` and `docs/20YYMMDD_product_requirements.md` must be created for all projects
   - Maintain an index of these documents in both CLAUDE.md and README.md
   - **AI INSTRUCTION**: Proactively create TODO.md and product_requirements.md if missing, then notify the user

#### TODO.md as Development Log

1. **IMPORTANT**: Maintain `docs/TODO.md` as central development log and task tracker
2. Include:
   - Current/upcoming tasks (with checkboxes)
   - Development decisions and rationale
   - Recurring issues and their solutions
   - Technical debt items
   - Code audit reminders
3. Update when: starting tasks, making progress, completing tasks, encountering issues
4. Check at start of each session

#### Session Start Checklist

1. Check `docs/TODO.md` for current state and development log
2. Run `git status` to see uncommitted changes
3. Run code tracking commands (see Maintenance section)
4. Review recent commits: `git log --oneline -5`
5. Check for outdated dependencies (if applicable)

#### Git Usage (Solo Developer)

1. Create git repo for each project with .gitignore
2. Work directly on main branch (no need for feature branches with multiple AI agents sharing same filesystem)
3. Conventional commit format: "type: Brief description" (fix, feat, docs, refactor, test, chore)
4. Before committing: run lint/build checks and verify functionality
5. Keep main branch stable and deployable
6. Commit regularly to track changes and enable rollback if needed

### Development Workflow

#### Planning and Implementation

1. Discuss approach and evaluate pros/cons before coding
2. Make small, testable incremental changes
3. Address code duplication proactively
4. When fixing issues, check for similar problems elsewhere
5. Document recurring issues in TODO.md

#### Code Quality Standards

1. Handle errors properly and validate inputs
2. Follow code conventions and established patterns
3. Never expose secrets/keys
4. Write self-documenting code with type safety
5. Remove debug output before production
6. **Documentation**: Be concise but accurate - avoid verbose explanations
   - Code comments only when necessary for complex logic
   - Documentation should be clear, direct, and to the point
   - Avoid redundant comments that merely describe what code already shows
   - Focus on "why" not "what" when documenting

#### Code Maintenance and Refactoring

1. **AI Tracking Instructions**:
   - Before each session: run `git diff --stat` to see recent changes
   - Track cumulative additions: `git log --numstat --pretty=format:'' | awk '{ add += $1 } END { print add }'`
   - When total additions exceed 500 lines since last audit, prompt user:
     "Added ~500+ lines since last code audit. Should we review for refactoring opportunities?"
   - Add to TODO.md: "Code audit pending (X lines added since DATE)"
2. Automatic refactoring triggers:
   - Duplicate code blocks (3+ occurrences)
   - Functions > 50 lines
   - Files > 300 lines
   - Multiple similar error handlers
3. During audits, check for:
   - Extractable shared utilities
   - Complex functions to split
   - Dead code to remove
   - Performance bottlenecks
4. Track findings in TODO.md under "Notes > Technical Debt"
5. Always refactor before major feature additions

### Debugging and Logging

1. **Use a proper logging framework** instead of console.log
2. Remove console.log statements before committing
3. Clean up debug output before production
4. Minimize linter suppressions
5. Use structured logging with appropriate log levels
6. For framework-specific logging tools, see the relevant section below

## Framework-Specific Guidelines

### Web/Frontend (If Applicable)

#### Technology Stack

1. Next.js for the framework
2. TypeScript for type safety
3. shadcn/ui for UI components
4. Tailwind CSS for styling
5. React for UI library

#### Component Patterns

1. Use server components by default
2. Add 'use client' only when necessary
3. Keep components focused on single responsibility
4. Extract complex logic to custom hooks
5. Follow container/presentational pattern where appropriate

#### State Management

1. React context for global state
2. Keep state local when possible

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dingran/ai-rules](https://github.com/dingran/ai-rules) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
