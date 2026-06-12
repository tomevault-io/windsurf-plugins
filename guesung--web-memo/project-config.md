---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 🚨 CRITICAL: Subagent (Task Tool) Usage Policy

**MANDATORY**: Before starting ANY non-trivial task, you MUST evaluate whether to use subagents (Task tool). This is NOT optional.

### When to Use Subagents (REQUIRED)

| Scenario | Required Subagent Type |
|----------|----------------------|
| Codebase exploration / "where is X?" | `Explore` |
| Multi-file search or pattern finding | `Explore` |
| Understanding code architecture | `Explore` |
| Implementation planning | `Plan` |
| Complex multi-step tasks | `general-purpose` |
| Code quality improvements | `refactoring-expert` |
| Frontend code refactoring | `frontend-refactor-kr` |
| Security review | `security-engineer` |
| Performance optimization | `performance-engineer` |
| System design decisions | `system-architect` |
| Creating PRs with proper workflow | `git-pr-workflow` |

### Execution Rules

1. **ALWAYS check subagent applicability FIRST** before doing any work yourself
2. **Explore agent is MANDATORY** for:
   - Any question about "where is X in the codebase?"
   - Any search across multiple directories
   - Understanding how a feature works
   - Finding all usages of a function/component
3. **Plan agent is MANDATORY** for:
   - Any feature implementation with 3+ steps
   - Architectural changes
   - Refactoring across multiple files
4. **Run subagents in PARALLEL** when possible to maximize efficiency
5. **DO NOT** manually search/grep when Explore agent can do it faster

### Anti-Patterns (NEVER DO THESE)

```
❌ Manually running multiple Grep/Glob commands to find code
   → Use Explore agent instead

❌ Starting implementation without planning for complex tasks
   → Use Plan agent first

❌ Running subagents sequentially when they could run in parallel
   → Launch multiple Task tools in single message

❌ Ignoring specialized agents (security, performance, etc.)
   → Match task type to appropriate agent
```

### Decision Flowchart

```
Task received
    │
    ├─ "Where is X?" or "Find all Y" → Explore agent (MANDATORY)
    │
    ├─ Complex implementation (3+ steps) → Plan agent (MANDATORY)
    │
    ├─ Code quality/refactoring → refactoring-expert or frontend-refactor-kr
    │
    ├─ Security concerns → security-engineer
    │
    ├─ Performance issues → performance-engineer
    │
    ├─ PR creation → git-pr-workflow
    │
    └─ Simple, single-file change → Do it yourself
```

**Remember**: Using subagents is FASTER and MORE ACCURATE than doing everything yourself. When in doubt, USE A SUBAGENT.

---

## Development Commands

**Package Manager**: This project uses `pnpm` (version 10.23.0) and requires Node.js >=22.17.0.

### Core Development Commands
```bash
# Install dependencies
pnpm i

# Development (all packages)
pnpm dev

# Development (extension only)
pnpm dev:extension

# Development (web app only) 
pnpm dev:web

# Production build
pnpm build

# Build extension only
pnpm build:extension

# Build web app only
pnpm build:web
```

### Quality & Testing
```bash
# Code formatting (Biome)
pnpm format

# Linting
pnpm lint
pnpm lint:fix

# Type checking
pnpm type-check

# Unit tests (Vitest)
pnpm test:jest

# E2E tests (Playwright)
pnpm test:e2e

# View E2E test report
pnpm test-report:e2e
```

### Extension Development
```bash
# Build extension for Firefox
pnpm build:extension -- --firefox

# Package extension for distribution
pnpm package

# Create distributable zip
pnpm zip

# Update version across all packages
pnpm update-version
```

### Database & Types
```bash
# Generate Supabase types
pnpm generate-supabase-type
```

## Project Architecture

### Monorepo Structure (Turborepo)

**Apps:**
- **`apps/chrome-extension/`** - Chrome Extension Manifest V3 core application
- **`apps/web/`** - Next.js 14.2.10 web application
- **`apps/mobile/`** - React Native/Expo mobile application

**Pages (Extension UI):**
- **`pages/`** - Extension UI pages (popup, side-panel, options, content-ui, devtools)

**Shared Packages:**
- **`packages/shared/`** - Shared utilities, hooks, types, and business logic
- **`packages/ui/`** - Shared UI components library (shadcn/ui based)
- **`packages/env/`** - Environment variable management
- **`packages/tailwind-config/`** - Shared TailwindCSS configuration
- **`packages/tsconfig/`** - Shared TypeScript configurations
- **`packages/vite-config/`** - Shared Vite build configuration
- **`packages/hmr/`** - Hot Module Replacement utilities for extension
- **`packages/zipper/`** - Extension packaging utilities
- **`packages/dev-utils/`** - Development utilities
- **`packages/supabase-edge-functions/`** - Supabase Edge Functions

**Testing & Infrastructure:**
- **`e2e/`** - Playwright end-to-end testing suite

### Core Technologies
- **Frontend**: React 18.3.1, TypeScript 5.5.3, TailwindCSS 3.4.x
- **State Management**: TanStack Query v5.59.0, React Hook Form 7.53.2
- **Build Tools**: Vite 5.3.3 (extension), Next.js 14.2.10 (web), Turbo 2.1.1
- **Backend**: Supabase (authentication, database, real-time)
- **Testing**: Playwright 1.47.0, Vitest 2.1.5
- **Code Quality**: Biome 2.0.0 (formatting/linting)

### Extension Architecture (Manifest V3)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [guesung/Web-Memo](https://github.com/guesung/Web-Memo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-12 -->
