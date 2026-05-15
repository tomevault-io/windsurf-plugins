---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

---

## What This Repository Is

**A reference library of Claude Code infrastructure** - NOT a working application.

This showcase contains production-tested skills, hooks, agents, and slash commands extracted from 6 months of real-world use managing TypeScript microservices. Users will ask you to help them integrate these components into their own projects.

**Critical:** When users ask to "add [component]", they mean copy it to THEIR project, not modify this showcase.

---

## Your Role: Integration Assistant

**Primary tasks:**
1. Help users integrate components from this showcase into their projects
2. Customize configurations for their specific project structure
3. Verify integrations work correctly
4. Explain how components function

**Before starting ANY integration:**
- Read `CLAUDE_INTEGRATION_GUIDE.md` - contains detailed integration instructions
- Ask about user's project structure
- Verify tech stack compatibility for skills
- Never assume directory structures

---

## Repository Architecture

```
.claude/
├── skills/                 # 5 production skills
│   ├── backend-dev-guidelines/     (Node.js/Express/Prisma)
│   ├── frontend-dev-guidelines/    (React/MUI v7/TanStack)
│   ├── skill-developer/            (Meta-skill, framework-agnostic)
│   ├── route-tester/               (JWT cookie auth testing)
│   ├── error-tracking/             (Sentry integration)
│   └── skill-rules.json            (Activation configuration)
├── hooks/                  # Rust-based hooks for automation
│   ├── RustHooks/                  (Rust implementation)
│   ├── skill-activation-prompt.sh  (ESSENTIAL - auto-activates skills)
│   ├── post-tool-use-tracker.sh    (ESSENTIAL - tracks file changes)
│   ├── tsc-check.sh                (Optional - monorepo TypeScript checks)
│   └── ... (other optional hooks)
├── agents/                 # 10 specialized agents (standalone)
│   ├── code-architecture-reviewer.md
│   ├── refactor-planner.md
│   ├── frontend-error-fixer.md
│   └── ... (7 more)
└── commands/               # 3 slash commands
    ├── dev-docs.md
    ├── dev-docs-update.md
    └── route-research-for-testing.md

dev/
└── README.md               # Dev docs pattern documentation
```

---

## Core Concepts

### 1. Skill Auto-Activation System

**The breakthrough feature** that solves "skills don't activate automatically":

**Components:**
- `skill-activation-prompt` hook (UserPromptSubmit)
- `post-tool-use-tracker` hook (PostToolUse)
- `skill-rules.json` configuration

**How it works:**
1. Hook runs on every user prompt
2. Checks skill-rules.json for trigger patterns (keywords, file paths, intent)
3. Automatically suggests relevant skills
4. Skills load only when needed

**These hooks work for any project without customization.**

### 2. Modular Skills (500-Line Rule)

Skills use progressive disclosure to avoid context limits:

```
skill-name/
  SKILL.md                  # <500 lines, overview + navigation
  resources/
    topic-1.md              # <500 lines each, deep dives
    topic-2.md
```

Claude loads main skill first, resources only when needed.

### 3. Dev Docs Pattern

Three-file structure for maintaining context across resets:

```
dev/active/[task-name]/
├── [task-name]-plan.md      # Strategic plan
├── [task-name]-context.md   # Current state, decisions
└── [task-name]-tasks.md     # Checklist
```

See `dev/README.md` for full pattern documentation.

---

## Rust Hook Implementation

**All hooks are implemented in Rust** for maximum performance and zero runtime dependencies.

### Performance Benefits:

| Metric | Rust Performance |
|--------|-----------------|
| **Startup Time** | ~2ms (60x faster than interpreted languages) |
| **Memory Usage** | 3-5MB (10x less than managed runtimes) |
| **Binary Size** | 1.8-2.4MB (self-contained) |
| **Runtime Required** | None (zero dependencies) |

### Installation Options:

**Recommended: Standalone Installation**

Linux / macOS:
```bash
# Build once, use everywhere
./install.sh
# Binaries installed to ~/.claude-hooks/bin/

# Or with SQLite support
./install.sh --sqlite
```

Windows:
```powershell
# Build once, use everywhere
.\install.ps1
# Binaries installed to %USERPROFILE%\.claude-hooks\bin\

# Or with SQLite support
.\install.ps1 -Sqlite
```

**Benefits:**
- ✅ Compile once (45s), use everywhere (0s per project)
- ✅ Update in one place, all projects benefit
- ✅ Tiny per-project footprint (50 bytes vs 2MB)
- ✅ Consistent version across all projects
- ✅ Idiomatic Cargo features (no multiple Cargo files)
- ✅ Works on Linux, macOS, and Windows

**See Complete Documentation:**
- `docs/rust-hooks.md` - Full implementation guide
- `docs/performance-comparison.md` - Performance analysis
- `docs/databases.md` - SQLite state management
- `docs/standalone-installation.md` - Installation details

---

## Integration Workflows

### When User Says: "Add [skill] to my project"

**Steps:**
1. **Ask clarifying questions:**
   - "What's your project structure? Monorepo or single app?"
   - "Where is your [backend/frontend] code located?"
   - For backend-dev-guidelines: "Do you use Express and Prisma?"

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dwalleck/catalyst](https://github.com/dwalleck/catalyst) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-08 -->
