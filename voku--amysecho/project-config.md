---
trigger: always_on
description: - **Mission:** Amy's Echo is a multimodal communication platform for non-verbal children focused on Deutsche Gebärdensprache (DGS) capture, training, and playback.
---

# Copilot Coding Agent Instructions for Amy's Echo

## Repository overview
- **Mission:** Amy's Echo is a multimodal communication platform for non-verbal children focused on Deutsche Gebärdensprache (DGS) capture, training, and playback.
- **Project Status:** This project is in a mature, pre-live product-hardening state. All major features for Phase 1, 2 and 3 have been implemented. The focus is now on optimization, bug fixing, and production readiness.
- **Source layout:**
  - `webapp/` React + TypeScript (Vite) UI with structured directories:
    - `src/components/` UI components
    - `src/hooks/` custom React hooks
    - `src/gesture/` gesture recognition pipeline
    - `src/training/` training queue and workflows
    - `src/services/` API clients and external services
    - `src/context/` React context providers
    - `src/types/` TypeScript type definitions
    - `src/utils/` utility functions
  - `server/` Node/TypeScript API plus Python training tools in `server/src/amyserver_tools/`.
  - `integration/` end-to-end tests that exercise the full training loop.
  - Docs live in `docs/` (start with `docs/planning/todo.md` for current priorities).

### Additional Resources
- **Development Workflow**: See `docs/workflows/development-workflow.md` for detailed Amy First development processes
- **Testing Strategy**: See `docs/testing/testing-strategy.md` for comprehensive testing guidelines
- **Current Status**: See `docs/planning/todo.md` for up-to-date implementation status

## 🚨 AMY FIRST DEVELOPMENT PRINCIPLES

**CRITICAL**: Every line of code must enhance Amy's ability to communicate. When in doubt, choose reliability over elegance, simplicity over features, and Amy's needs over technical metrics.

### Amy First Commitments
- ✅ **Zero interruption** - Amy's communication never pauses
- ✅ **Zero confusion** - Simple, clear UI always
- ✅ **Zero delay** - Instant feedback for everything
- ✅ **Zero failure** - Explicit recovery paths for communication-critical runtime failures
- ✅ **Zero judgment** - Celebrate attempts, not just success
- ✅ **Zero compromise** - Amy's needs come first

### Pre-Implementation Checklist
**Complete ALL items before writing code:**
- [ ] **Read `docs/planning/todo.md` completely** - Understand Amy's needs
- [ ] **Identify the "Amy Impact"** - How does this help Amy communicate?
- [ ] **Check existing implementation** - Don't duplicate work
- [ ] **Verify against Amy First principles** - Does this enhance communication?
- [ ] **Test current functionality** - Ensure nothing breaks
- [ ] **Document the "why"** - Explain how this serves Amy

## AI Assistant Workflow

**IMPORTANT**: Follow this step-by-step approach:

### 1. Discovery Phase (ALWAYS do this first)
- **Read the `docs/planning/todo.md` or task description completely** to understand the current priorities.
- **Review the existing documentation** in the `docs/` directory to understand the project's architecture and features.
- **Examine the existing codebase structure** using `find` or `ls` commands.
- **Study similar existing files** - look for patterns, naming conventions, and architectural decisions.
- **Run the test suite** to understand current functionality and ensure nothing is broken.
- **Check dependencies and configuration files** (`package.json`, `tsconfig.json`, etc.)

### 2. Planning Phase (Before any implementation)
- **Create a detailed implementation plan** that explains:
  - Which files need to be created/modified
  - What existing patterns you'll follow
  - How your changes integrate with current architecture
  - What tests need to be added/updated
- **Identify potential breaking changes** and migration steps. The project is not live yet, so prefer migrating callers, fixtures, and docs to the current contract over adding legacy fallback paths for obsolete internal shapes.
- **Plan your testing approach** - don't just implement features, plan how to verify they work

### 3. Implementation Phase
- **Start with tests** when adding new functionality (TDD approach)
- **Make small, incremental changes** - don't implement everything at once
- **Follow existing code patterns exactly** - don't introduce new architectural concepts without justification
- **Test continuously** - run relevant tests after each significant change
- **Use German for all user-facing text and any error messages that Amy sees in the app.** Developer-facing logs, console output, and internal identifiers can remain in English.
- **Update the documentation** to reflect your changes. This includes the `docs/` directory and any relevant `README.md` files.

### 4. Verification Phase (MANDATORY)
- **Run the full test suite** - all tests must pass
- **Verify type checking** - no TypeScript errors
- **Test the actual functionality** - don't assume it works because tests pass
- **Check for integration issues** - ensure your changes work with existing features

## Build and test commands (run from repo root)

> **Note:** You can also run `./scripts/full-check.sh` to execute most of the checks below.
- **Webapp:**
  - Install: `npm ci --prefix webapp`
  - Type check: `npm run type-check --prefix webapp`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [voku/AmysEcho](https://github.com/voku/AmysEcho) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
