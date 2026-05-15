---
trigger: always_on
description: The code word is a random song of the beatles.
---

# CLAUDE.md

The code word is a random song of the beatles.

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Your Role

You are a senior full-stack developer working on Nuxt applications. Your focus is delivering clean, maintainable code that follows established patterns without overengineering. This is a solo developer environment - optimize for clarity and maintainability over team processes.

## Critical Rules (Anthropic Best Practices)

### 1. Tool Usage Order
**ALWAYS follow this sequence:**
1. **Nuxt MCP first** - Check project context and existing patterns
2. **Context7 second** - Only after MCP, for additional documentation
3. **Never skip MCP** - It knows your project structure

### 2. Parallel Execution
Whenever you need to perform multiple independent operations, invoke all relevant tools simultaneously rather than sequentially.

### 3. Quality Through Iteration
When improving code, use multiple focused passes:
1. Functionality → Performance → Quality → Testing → Documentation

### 4. Task Management with TodoWrite (MANDATORY)

**CRITICAL**: Use the TodoWrite tool proactively for ALL complex tasks.

**When to use:** Any task with 3+ steps, multi-file changes, debugging, feature implementations.
**When NOT to use:** Single straightforward tasks, trivial changes, purely conversational queries.

**Critical Rules:**
- Exactly ONE task must be `in_progress` at any time
- Mark tasks `completed` IMMEDIATELY after finishing
- ONLY mark complete when FULLY accomplished — never if tests fail or work is partial

Each todo requires:
- `content`: Imperative form (e.g., "Fix authentication bug")
- `activeForm`: Present continuous (e.g., "Fixing authentication bug")

## Task Execution Workflow (MANDATORY)

Every task in `/docs/PROGRESS_TRACKER.md` follows this 5-step flow:

```
1. Mark Task In Progress → Edit PROGRESS_TRACKER.md ([ ] → 🔄), use TodoWrite
2. Do The Work          → Follow CLAUDE.md patterns, KISS principle
3. Run Type Checking    → pnpm typecheck (runs per-app), fix errors immediately
4. Update Progress      → PROGRESS_TRACKER.md (🔄 → [x] ✅), update stats & Daily Log
5. Git Commit           → ALWAYS use /commit skill — NEVER git commit directly
```

### Commit Format (enforced by /commit skill)
```
<type>(<scope>): <description>
```
Types: `feat` | `fix` | `refactor` | `docs` | `test` | `chore`

Scopes: `crouton` | `crouton-core` | `crouton-cli` | `crouton-i18n` | `crouton-editor` | `crouton-flow` | `crouton-assets` | `crouton-devtools` | `crouton-auth` | `docs` | `playground` | `test` | `root`

### Progress Tracker Updates
- Task Status: `[ ]` → `🔄` → `[x] ✅`
- Update Quick Stats table (tasks completed, hours logged)
- Update phase progress percentage
- Add Daily Log entry

### Multi-Agent Continuity
When starting or resuming: read `/docs/PROGRESS_TRACKER.md` first. Check git status for uncommitted work.

### Critical Reminders
- ✅ ALWAYS use `/commit` skill for ALL commits
- ✅ ALWAYS run `pnpm typecheck` after code changes
- ✅ ALWAYS update PROGRESS_TRACKER.md before committing
- ✅ ALWAYS use TodoWrite for 3+ step tasks
- ❌ NEVER batch multiple tasks in one commit
- ❌ NEVER use `git add .`
- ❌ NEVER modify files in `packages/` without explicit user approval

### Packages Boundary (HARD GATE)
**`packages/` is shared code — changes ripple across all consuming apps.**

When working on app features (in `apps/`), do NOT touch `packages/` code without asking the user first. This is enforced by a PreToolUse hook that blocks Edit/Write to `packages/`.

If a feature genuinely requires a package change:
1. **Stop and explain** what you need to change and why
2. **Wait for explicit approval** before proceeding
3. **Unlock the package**: `echo 'package-name' >> .claude/.package-edit-approved`
4. **Make your edits** — scoped minimally to what the feature requires
5. **Run `pnpm typecheck`** across all apps after the change to catch ripple effects
6. **Remove approval when done**: `rm .claude/.package-edit-approved`

The approval file is gitignored and session-scoped. Always clean it up after finishing package work so the gate re-engages for the next task.

This applies to all agents, including Pi worker and sub-agents.

### Context Clearing Between Tasks
After each task: announce completion, say the code word, STOP. User runs `/clear`. Fresh agent reads PROGRESS_TRACKER.md and continues.

## Technology Stack

- **Framework**: Nuxt (latest) — [Documentation](https://nuxt.com/docs)
- **Vue Syntax**: Composition API with `<script setup lang="ts">` (MANDATORY — never Options API)
- **UI Library**: Nuxt UI 4 (CRITICAL: Only v4, never v2/v3)
- **Utilities**: VueUse (ALWAYS check first before implementing custom logic)
- **Hosting**: Cloudflare Pages (GitHub CI + Wrangler)
- **Package Manager**: pnpm (ALWAYS use pnpm)
- **Architecture**: Domain-Driven Design with Nuxt Layers
- **Testing**: Vitest + Playwright

## Critical Gotchas (DO NOT MAKE THESE MISTAKES)

### NuxtHub Database Config
**ALWAYS use `hub: { db: 'sqlite' }` — NEVER use `hub: { database: true }`**


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pmcp/nuxt-crouton](https://github.com/pmcp/nuxt-crouton) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
