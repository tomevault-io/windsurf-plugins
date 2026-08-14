---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## HIGH PRIORITY

- **IF YOU DON'T CHECK SKILLS** your task will be invalidated and we will generate rework
- **YOU CAN ONLY** finish a task if `make check` passes at 100% (runs `fmt + lint-fix + typecheck + test`). No exceptions — failing any of these commands means the task is **NOT COMPLETE**
- **`bun run lint` treats warnings as errors**. **Zero warnings allowed** — any oxlint warning is a blocking failure, not something to ignore
- **ALWAYS** check dependent file APIs before writing tests to avoid writing wrong code
- **NEVER** use workarounds, especially in tests — always use the `no-workarounds` skill for any fix/debug task + `test-antipatterns` for tests
- **ALWAYS** use the `no-workarounds` and `systematic-debugging` skills when fixing bugs or complex issues
- **YOU MUST** use Context7 or Exa (`exa-web-search-free` skill via mcporter) when researching external libraries/frameworks before implementing integrations — always do **3-7 searches** with Exa for better results
- **NEVER** use Context7 or Exa to search local project code — for local code, use Grep/Glob instead
- **YOU SHOULD NEVER** install dependencies by hand in `package.json` without verifying the package exists and checking its latest version — always use `bun add` instead

## MANDATORY REQUIREMENTS

- **MUST** run `make check` (or equivalently `bun run format && bun run lint && bun run typecheck && bun run test`) before completing ANY subtask. All commands must exit with **zero errors and zero warnings**. If any command fails, fix the issues and re-run until all pass
- **ALWAYS USE** the `typescript-advanced` skill before working with complex TypeScript patterns
- **ALWAYS USE** the `vitest` skill before writing or modifying tests
- **ALWAYS USE** the `zod` skill when working with validation schemas
- **Skipping any verification check will result in IMMEDIATE TASK REJECTION**

## Skills Enforcement

When working on this project, **always use the relevant skills** for the technology being touched:

### Generator & CLI

- **TypeScript patterns**: Use `typescript-advanced` skill
- **Validation (Zod schemas)**: Use `zod` skill
- **Utility functions (es-toolkit)**: Use `es-toolkit` skill
- **Testing (Vitest)**: Use `vitest` skill

### Template Content (what gets generated)

When editing templates that will be part of generated projects, use the domain-specific skill:

- **React/Frontend templates**: Use `react` skill
- **Hono/Backend templates**: Use `hono` skill
- **Database templates (Drizzle)**: Use `postgres-drizzle` + `drizzle-orm` skills
- **Auth templates (Better Auth)**: Use `better-auth-best-practices` skill
- **Organization templates**: Use `organization-best-practices` skill
- **Stripe templates**: Use `stripe-integration` + `stripe-best-practices` skills
- **Stripe subscription templates**: Use `stripe-subscriptions` skill
- **Stripe webhook templates**: Use `stripe-webhooks` skill
- **Inngest templates**: Use `inngest` skill
- **Storybook templates**: Use `storybook` skill
- **UI components (shadcn/ui)**: Use `shadcn` skill
- **TanStack Router**: Use `tanstack-router-best-practices` skill
- **TanStack Query**: Use `tanstack-query-best-practices` skill
- **Zustand**: Use `zustand` skill

### Process & Quality

- **Before any creative/feature work**: Use `brainstorming` skill
- **Executing implementation plans**: Use `executing-plans` skill
- **Debugging/fixing bugs**: Use `no-workarounds` + `systematic-debugging` skills (enforce root-cause fixes)
- **Writing/changing tests**: Use `test-antipatterns` skill (prevents mock-testing-mocks and production pollution)
- **Before claiming task is complete**: Use `verification-before-completion` skill
- **Code review (cross-model)**: Use `adversarial-review` skill
- **Architectural analysis/dead code**: Use `architectural-analysis` skill
- **PR review fixes**: Use `fix-coderabbit-review` skill
- **Git rebase/conflicts**: Use `git-rebase` skill
- **Prompt generation for LLMs**: Use `to-prompt` skill
- **Code analysis (Pal MCP)**: Use `pal` skill
- **Discover/install skills**: Use `find-skills` skill
- **Creating skills**: Use `skills-best-practices` skill

## Commands

```bash
# Setup (after cloning)
git submodule update --init --recursive  # Initialize skills submodule

# Development
bun run dev              # Run CLI in development mode

# Quality
bun run lint             # Format (oxfmt) + lint (oxlint)
bun run typecheck        # Type check with tsc
bun run format           # Format with oxfmt
bun run format:check     # Check formatting
bun run test             # Run tests (Vitest)

# Makefile shortcuts
make check               # Run format + lint-fix + typecheck + test
make commit              # Run check pipeline + git add + opencommit
make clean               # Remove node_modules, build artifacts, caches
make update              # Interactive dependency update (taze)
```

## CRITICAL: Git Commands Restriction


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pedronauck/devstack](https://github.com/pedronauck/devstack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-14 -->
