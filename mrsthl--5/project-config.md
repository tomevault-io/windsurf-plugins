---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is the **5-Phase Workflow** package - a systematic, AI-assisted feature development workflow for Claude Code. It's an npm package that installs commands, agents, skills, and hooks into `.claude/` directories to enable structured feature development.

**Key Concept:** This is NOT a traditional application to run/build. It's an installer that copies workflow files to users' projects. The workflow files (commands, agents, skills) are written in Markdown and consumed by Claude Code.

---

### ⚠️ CRITICAL DEVELOPMENT RULE

**ALWAYS check and update `bin/install.js` when making changes to workflow files!**

When you add, rename, or remove ANY file in:
- `src/commands/5/`
- `src/skills/`
- `src/hooks/`
- `src/templates/`

You **MUST** update the `getWorkflowManagedFiles()` function in `bin/install.js` to include it in the selective update system.

Always ensure the changes are compatible with **Claude and Codex**.

---

## Commands for Development

### Installation Testing

```bash
# Test local installation (in another project directory)
node bin/install.js

# Test with different options
node bin/install.js --global
node bin/install.js --uninstall
node bin/install.js --upgrade
node bin/install.js --check     # Check if update is available
```

### Package Testing

```bash
# Test the package locally via npx
npm link
npx 5-phase-workflow

# Or test directly
npm pack
# This creates a .tgz file you can install elsewhere
```

### Testing

```bash
npm test              # Run all tests
npm run test:install  # Verify install.js file list matches src/
npm run test:hook     # Test check-updates hook
npm run test:update   # Test update system
```

There is no build step. The workflow files are static Markdown copied during installation.

## Architecture

### Directory Structure

```
src/
├── commands/5/              # User-facing workflow commands
│   ├── configure.md         # Initial project setup
│   ├── reconfigure.md       # Re-run configuration
│   ├── plan-feature.md      # Phase 1: Feature planning
│   ├── plan-implementation.md # Phase 2: Implementation planning
│   ├── implement-feature.md # Phase 3: Orchestrated implementation
│   ├── verify-implementation.md # Phase 4: Verification
│   ├── review-code.md       # Phase 5: Code review
│   ├── address-review-findings.md # Fix review findings
│   ├── discuss-feature.md   # Feature discussion
│   ├── quick-implement.md   # Lightweight implementation
│   ├── unlock.md            # Unlock stuck workflows
│   └── update.md            # Update workflow version
│
├── skills/                  # Atomic operations
│   ├── build-project/
│   ├── run-tests/
│   ├── configure-docs-index/
│   ├── configure-skills/
│   └── generate-readme/
│
├── templates/               # Output templates
│   ├── workflow/            # Workflow output templates (PLAN.md, STATE.json, etc.)
│   ├── ARCHITECTURE.md      # Project documentation templates
│   ├── CONCERNS.md
│   ├── CONVENTIONS.md
│   ├── INTEGRATIONS.md
│   ├── STACK.md
│   ├── STRUCTURE.md
│   └── TESTING.md
│
├── hooks/
│   ├── statusline.js        # Status line integration
│   ├── check-updates.js     # Update notifications
│   ├── check-reconfig.js    # Reconfiguration prompts
│   ├── config-guard.js      # Configuration enforcement
│   └── plan-guard.js        # Plan phase enforcement
│
└── settings.json            # Claude Code settings

bin/
└── install.js               # Main installer script
```

### The 5-Phase Workflow

1. **Feature Planning** (`/5:plan-feature`)
   - Intensive Q&A (5-10 questions) to understand requirements
   - Challenges assumptions, explores edge cases
   - Creates feature spec at `.5/features/{ticket-id}/feature.md`

2. **Implementation Planning** (`/5:plan-implementation`)
   - Quick codebase scan to understand structure
   - Asks 2-3 technical questions
   - Creates simple plan at `.5/features/{ticket-id}/plan.md`
   - Plan describes WHAT to build, not complete code

3. **Orchestrated Implementation** (`/5:implement-feature`)
   - Reads plan.md
   - Spawns agents for each step (instructions embedded inline)
   - Agents explore codebase to find patterns
   - State tracked in `.5/features/{ticket-id}/state.json`

4. **Verify Implementation** (`/5:verify-implementation`)
   - Checks files exist
   - Runs build and tests
   - Generates verification report

5. **Code Review** (`/5:review-code`)
   - Runs CodeRabbit CLI (optional)
   - Categorizes findings
   - Applies approved fixes

**Context Management:** Running `/clear` between phases is optional. Phase 1→2 benefits from keeping context (plan-implementation detects live context and skips redundant steps). For Phase 2→3 and later, `/clear` is recommended to free context for implementation agents. Each phase is designed to work both with and without prior context.

### Key Design Patterns

#### Simple Plan Format

Phase 2 creates a single `plan.md` file:

```markdown
---
ticket: PROJ-1234
feature: PROJ-1234-add-schedule
created: 2026-01-28T10:00:00Z
---

# Implementation Plan: PROJ-1234

Add emergency schedule tracking.

## Components


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mrsthl/5](https://github.com/mrsthl/5) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
