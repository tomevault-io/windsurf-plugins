---
trigger: always_on
description: This file provides guidance to AI when working with code in this repository.
---

This file provides guidance to AI when working with code in this repository.

# Development Partnership

We're building production-quality code together. Your role is to create maintainable, efficient solutions while catching potential issues early.

When you seem stuck or overly complex, I'll redirect you - my guidance helps you stay on track.

## 📚 Technical Rules Reference

This document provides general guidance, workflow, and project context. For detailed technical rules:

- **Backend Development**: See `.cursor/rules/backend.mdc` for comprehensive backend rules including:
  - 4-layer Clean Architecture patterns
  - Type organization (General Types vs Route-Specific Types)
  - Database patterns (postgres.js usage, query patterns, transactions)
  - Naming conventions, import organization, error handling
  - Route implementation patterns, security requirements, testing requirements

- **Frontend Development**: See `.cursor/rules/frontend-rule.mdc` for comprehensive frontend rules including:
  - Component architecture and organization
  - Styling guidelines (Tailwind CSS v4, shadcn/ui)
  - Performance and state management patterns
  - Testing strategy, TypeScript best practices
  - Asset handling, development workflow

**Always check these rule files before implementing backend or frontend features** to ensure compliance with detailed technical standards.

## 🚨 AUTOMATED CHECKS ARE MANDATORY

**ALL lint/test issues are BLOCKING - EVERYTHING must be ✅ GREEN!**  
No errors. No formatting issues. No linting problems. Zero tolerance.  
These are not suggestions. Fix ALL issues before continuing.

## CRITICAL WORKFLOW - ALWAYS FOLLOW THIS!

### Research → Plan → Implement

**NEVER JUMP STRAIGHT TO CODING!** Always follow this sequence:

1. **Research**: Explore the codebase, understand existing patterns
2. **Plan**: Create a detailed implementation plan and verify it with me
3. **Implement**: Execute the plan with validation checkpoints

When asked to implement any feature, you'll first say: "Let me research the codebase and create a plan before implementing."

For complex architectural decisions or challenging problems, use **"ultrathink"** to engage maximum reasoning capacity. Say: "Let me ultrathink about this architecture before proposing a solution."

### USE MULTIPLE AGENTS!

_Leverage subagents aggressively_ for better results:

- Spawn agents to explore different parts of the codebase in parallel
- Use one agent to write tests while another implements features
- Delegate research tasks: "I'll have an agent investigate the database schema while I analyze the API structure"
- For complex refactors: One agent identifies changes, another implements them

Say: "I'll spawn agents to tackle different aspects of this problem" whenever a task has multiple independent parts.

### Reality Checkpoints

**Stop and validate** at these moments:

- After implementing a complete feature
- Before starting a new major component
- When something feels wrong
- Before declaring "done"
- **WHEN LINTING/TESTS FAIL WITH ERRORS** ❌

Run: `bun run format && bun test && bun run lint`

> Why: You can lose track of what's actually working. These checkpoints prevent cascading failures.

### 🚨 CRITICAL: Lint/Test Failures Are BLOCKING

**When linting or tests report ANY issues, you MUST:**

1. **STOP IMMEDIATELY** - Do not continue with other tasks
2. **FIX ALL ISSUES** - Address every ❌ issue until everything is ✅ GREEN
3. **VERIFY THE FIX** - Re-run the failed command to confirm it's fixed
4. **CONTINUE ORIGINAL TASK** - Return to what you were doing before the interrupt
5. **NEVER IGNORE** - There are NO warnings, only requirements

This includes:

- Formatting issues (prettier, eslint --fix)
- Linting violations (eslint, typescript compiler)
- Test failures (bun test)
- Type checking errors (bun run check-types)
- ALL other checks

Your code must be 100% clean. No exceptions.

**Recovery Protocol:**

- When interrupted by a lint/test failure, maintain awareness of your original task
- After fixing all issues and verifying the fix, continue where you left off
- Use the todo list to track both the fix and your original task

## Working Memory Management

### When context gets long:

- Re-read this CLAUDE.md file
- Summarize progress in a PROGRESS.md file
- Document current state before major changes

### Maintain TODO.md:

```
## Current Task
- [ ] What we're doing RIGHT NOW

## Completed
- [x] What's actually done and tested

## Next Steps
- [ ] What comes next
```

## JavaScript/TypeScript-Specific Rules

### FORBIDDEN - NEVER DO THESE:

- **NO `any` types** - use proper TypeScript types!
- **NO `setTimeout()` for synchronization** - use Promises and async/await!
- **NO** keeping old and new code together
- **NO** migration functions or compatibility layers
- **NO** versioned function names (processV2, handleNew)
- **NO** complex error class hierarchies
- **NO** TODOs in final code
- **NO** `console.log` in production code - use proper logging

### Required Standards:

- **Delete** old code when replacing it
- **Meaningful names**: `userId` not `id`
- **Early returns** to reduce nesting
- **Proper types**: Use TypeScript interfaces and types consistently

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [naufaldi/viralkan-app](https://github.com/naufaldi/viralkan-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
