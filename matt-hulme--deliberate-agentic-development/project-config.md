---
trigger: always_on
description: **FREEMODE (Context/Workflow Control):**
---

# Agent Workflow System

**FREEMODE (Context/Workflow Control):**

FREEMODE allows you to temporarily disable all workflow rules and context loading when you need a lightweight conversation without the full agent system.

**Usage:**
- `FREEMODE START` - Early return from AGENTS.md. Ignore all workflow rules and context loading. Treat the chat like a blank slate.
- `FREEMODE END` - Load AGENTS.md from the beginning and follow all standard workflow rules as normal.

**When to use FREEMODE:**
- Quick questions that don't require project context
- Debugging or exploration outside the workflow
- When context loading becomes too heavy (30k+ tokens)
- Experimentation without affecting state.json

**Note:** FREEMODE does not modify state.json. Your project position is preserved.

## Table of Contents

1. [Context Loading Instructions](#1-context-loading-instructions)
2. [Required Tools & Configuration](#2-required-tools--configuration)
3. [Workflows](#3-workflows)
4. [Work Unit Hierarchy](#4-work-unit-hierarchy)
5. [State Management](#5-state-management)
6. [Communication Patterns](#6-communication-patterns)
7. [Documentation Structure](#7-documentation-structure)
8. [Template Notation System](#8-template-notation-system)

---

## 1. Context Loading Instructions

**File Location Note:** All workflow and documentation files referenced below are located in the `.agents/` directory unless otherwise specified.

**MANDATORY FIRST STEPS - EXECUTE IN ORDER (skip if FREEMODE active):**

1. **Check Project State**
   - Read state.json to check workflow phase
   - If no state.json exists:
     a. First time setup → Ask: "Tell me about your project"
     b. Review Required Tools (Section 2) and verify connections
     c. Missing tools? → Configure required MCPs and CLI tools
   - Otherwise continue with workflow phase from state.json

2. **Load Core Documentation**
   - Always load PRODUCT-OVERVIEW.md for product vision and tech stack

3. **Load Phase-Specific Workflow**

   Based on `project_phase` from state.json:
   - `"planning-project"` → Load PLAN.md then PLAN-PROJECT.md
   - `"planning-issue"` → Load PLAN.md then PLAN-ISSUE.md
   - `"implementing"` → Load IMPLEMENT.md

4. **Load Rules & System Documentation**

   **Decision Tree:**
   ```
   Check project_phase from state.json
   │
   ├─ "planning-project"
   │  └─ Load ALL files in rules/
   │     (Architecture decisions need full context)
   │
   └─ "planning-issue" OR "implementing"
      └─ Smart loading based on current work:
         a. List filenames in rules/ and documentation/systems/
         b. Infer relevance from filename (don't search content)
         c. Load only what's relevant to current work

   Example: Task "Create user API"
   → Load: API-RULES.md, AUTHENTICATION.md
   → Skip: UI-RULES.md, TESTING.md (unless relevant)
   ```

   **Important:** Don't assume specific file names - work with what's there

5. **Load Agent-Specific Config** (if exists)
   - Claude → .claude/
   - Cursor → .cursor/
   - Codex → .codex/

---

## 2. Required Tools & Configuration

*Note: This section defines agent tooling requirements only. Document your tech stack in PRODUCT-OVERVIEW.md.*

### Core Configuration
```
PM_TOOL=Linear                # Project management tool
GIT_PLATFORM=GitHub           # Git platform
TICKET_PREFIX=XXX             # Ticket prefix (change to your prefix)
PROJECT_NAME=your-project     # Project name (change to your project)
PROJECT_ROOT=/path/to/project # Project root (change to your path)
```

### Required Tools & MCPs

#### Essential Command Line Tools
- **Git** - Version control
- **GitHub CLI** (`gh`) - Pull request and repository management

#### Required MCPs
- **Linear MCP** - Issue tracking and project management (or your configured PM_TOOL MCP)

**Note:** Build tools, language runtimes, testing frameworks, and other project-specific tooling should be documented in `.agents/documentation/PRODUCT-OVERVIEW.md` under the Tech Stack section, not here.

### Naming Conventions

#### Branches
- **Single PR:** `{{TICKET_PREFIX}}-XXX` (e.g., `{{TICKET_PREFIX}}-123`)
- **Multi-part:** `{{TICKET_PREFIX}}-XXX-pt-2`, `{{TICKET_PREFIX}}-XXX-pt-3`

#### Commits
```
Format: M[milestone].[issue].[task] - Task description
Example: M1.2.3 - Add user authentication
```

#### Pull Requests
```
Format: M[milestone].[issue] - [Issue Name] ({{TICKET_PREFIX}}-XXX)
Example: M1.2 - API Layer ({{TICKET_PREFIX}}-101)
Note: {{TICKET_PREFIX}}-XXX included for Linear linkage
```

---

## 3. Workflows

### 3.1 🚨 MANDATORY WORKFLOW COMPLIANCE 🚨

- Follow sequential workflow steps in exact order - NO skipping, combining, or reordering
- Use templates exactly as provided
- One task = One commit = One review (never bundle)
- Get user approval at all CHECKPOINT steps

**If Confused:** Check state.json, then reload appropriate workflow document

### 3.2 Two Workflow Phases

**PLANNING Phase (PLAN.md + sub-workflows):**
- Project Planning (PLAN-PROJECT.md): One-time setup, vision, tech stack, all milestones
- Issue Planning (PLAN-ISSUE.md): Per-milestone task breakdown (all milestones before implementation)

**IMPLEMENTING Phase (IMPLEMENT.md):**
- Execute tasks one-by-one with reviews
- Create PRs and merge when issues complete

## 4. Work Unit Hierarchy

### 4.1 Work Unit Definitions

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Matt-Hulme/deliberate-agentic-development](https://github.com/Matt-Hulme/deliberate-agentic-development) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
