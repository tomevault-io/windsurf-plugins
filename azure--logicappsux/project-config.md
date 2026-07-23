---
trigger: always_on
description: <!-- AUTO-GENERATED from docs/ai-setup/. DO NOT EDIT directly. Run: pnpm run ai:generate -->
---

<!-- AUTO-GENERATED from docs/ai-setup/. DO NOT EDIT directly. Run: pnpm run ai:generate -->

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.


# Logic Apps UX — AI Assistant Guide

## Always Do These First (every task, no exceptions)

These rules apply to **every** task — including ones that look trivial. Skipping them because a task "looks like a one-liner" is a known failure mode.

1. **Check repo git aliases before running any `git` command.**
   - Run `git config --get-regexp '^alias\.'` and prefer repo aliases over raw `git` invocations.
   - Examples in this repo: `git new <branch>` creates a worktree at `../<branch>` from `origin/main` and overlays `.github` / `.squad` with skip-worktree. Do **not** hand-roll `git worktree add` for new branches.
2. **Scan the knowledge index before planning or acting.**
   - Read `.squad/knowledge/INDEX.md` and open any listed file whose triggers match the task domain (git/worktree, e2e, CI, reviews, customer repro, unit tests, etc.).
   - For non-trivial work also follow the chief-engineer charter's subagent workflow.
3. **Never skip steps 1–2 because the task "looks trivial."** Trivial-looking requests (worktree creation, branch setup, running a single test) are exactly where repo-specific conventions get missed.

## Repository Overview

Azure Logic Apps UX is a monorepo containing the UI components for Azure Logic Apps visual workflow designer. It powers the designer experience across Azure Portal, VS Code extension, Power Automate, and standalone environments.

## Essential Commands

### Development
```bash
pnpm install              # Install all dependencies (run from root)
pnpm run start            # Start standalone designer (https://localhost:4200)
pnpm run start:arm        # Start with Azure ARM authentication (generates token then starts)
```

### Building
```bash
pnpm run build            # Build all packages
pnpm run build:lib        # Build libraries only (for NPM publishing)
pnpm run build:extension  # Build VS Code extension
```

### Testing
```bash
# Unit tests
pnpm run test:lib              # Run all unit tests
pnpm run test:iframe-app       # IFrame chat app unit tests
pnpm run test:extension-unit   # VS Code extension unit tests

# Run a single unit test file
pnpm vitest run path/to/file.spec.ts

# Run tests for a specific package
pnpm --filter @microsoft/logic-apps-designer test -- <test-name>

# E2E tests (Playwright)
pnpm run e2e:setup             # One-time: install Playwright browsers
pnpm run test:e2e              # Run all E2E tests
pnpm run test:e2e --grep @mock # Run mock-API tests only
pnpm run test:e2e:ui           # Open E2E test UI (visual debugging)
pnpm run testgen               # Generate E2E test code

# VS Code extension E2E (ExTester)
pnpm run vscode:designer:e2e:ui       # UI mode
pnpm run vscode:designer:e2e:headless # Headless mode
```

### VS Code Extension E2E Tests (ExTester)
```bash
cd apps/vs-code-designer
npx tsup --config tsup.e2e.test.config.ts
node out/test/run-e2e.js

# Run specific phases via E2E_MODE env var
E2E_MODE="createonly"    # Phase 4.1: workspace creation
E2E_MODE="designeronly"  # Phase 4.2: designer lifecycle
E2E_MODE="newtestsonly"  # Phases 4.3-4.6: new tests

# CI matrix shard modes (each runs on its own GitHub Actions runner):
E2E_MODE="independentonly"      # 4.0 + 4.8b (no Phase 4.1 dep)
E2E_MODE="createplusdesigner"   # 4.1 → 4.2, 4.7
E2E_MODE="createplusnewtests"   # 4.1 → 4.3-4.6
E2E_MODE="createplusconversion" # 4.1 → 4.8a, 4.8c, 4.8d, 4.8e
```

Key knowledge files for E2E tests:
- `apps/vs-code-designer/src/test/ui/SKILL.md` — Complete learning document (700+ lines)
- `apps/vs-code-designer/src/test/ui/designerHelpers.ts` — Shared designer test helpers
- `apps/vs-code-designer/src/test/ui/runHelpers.ts` — Shared debug/run test helpers
- `apps/vs-code-designer/src/test/ui/run-e2e.ts` — Test launcher (7 phases)

### Code Quality
```bash
pnpm run check           # Format and lint with Biome (biome check --write .)
pnpm run extract         # Extract i18n strings
pnpm run compile:loc     # Compile localization strings
eslint --cache --fix     # Run ESLint
```

### VS Code Extension
```bash
pnpm run vscode:designer:pack  # Package VS Code extension
```

### Other Utilities
```bash
pnpm run generateArmToken  # Generate Azure ARM token for live API testing
pnpm run templates         # Download workflow templates
```

## Architecture Overview

### Technology Stack
- **Frontend**: React 18, TypeScript, Redux Toolkit, React Query (TanStack Query)
- **UI Libraries**: Fluent UI v8/v9, Monaco Editor, React Flow (XY Flow)
- **Build Tools**: PNPM workspaces, Turborepo, Vite, tsup
- **Testing**: Vitest (unit), Playwright (E2E)

### Repository Structure
```
/apps
  /Standalone         - Development environment / test harness (Vite + React)
  /docs               - Documentation site (Docusaurus)
  /iframe-app         - A2A Chat iframe application
  /vs-code-designer   - VS Code extension host
  /vs-code-react      - React views for VS Code webviews

/libs
  /a2a-core           - A2A protocol chat client SDK
  /chatbot            - AI chatbot integration

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Azure/LogicAppsUX](https://github.com/Azure/LogicAppsUX) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
