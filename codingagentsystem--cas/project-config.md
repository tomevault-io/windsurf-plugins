---
trigger: always_on
description: <!-- CAS:BEGIN - This section is managed by CAS. Do not edit manually. -->
---

<!-- CAS:BEGIN - This section is managed by CAS. Do not edit manually. -->
# IMPORTANT: USE CAS FOR TASK AND MEMORY MANAGEMENT

**DO NOT USE BUILT-IN TOOLS (TodoWrite, EnterPlanMode) FOR TASK TRACKING.**

Use CAS MCP tools instead:
- `mcp__cas__task` with action: create - Create tasks (NOT TodoWrite)
- `mcp__cas__task` with action: start/close - Manage task status
- `mcp__cas__task` with action: ready - See ready tasks
- `mcp__cas__memory` with action: remember - Store memories and learnings
- `mcp__cas__search` with action: search - Search all context

CAS provides persistent context across sessions. Built-in tools are ephemeral.
<!-- CAS:END -->

# CAS - Coding Agent System

Unified context system for AI agents: persistent memory, tasks, rules, and skills across sessions.

**Design Philosophy**: CAS is built for AI agents as the primary users. Humans are observers who review agent activity, provide feedback, and guide direction - but the tools and workflows are optimized for agent consumption and production.

## Use CAS for Task & Memory Management

**Agents use CAS MCP tools instead of built-in TodoWrite:**

```
mcp__cas__task action=create     - Create/track tasks
mcp__cas__task action=start      - Start working on a task (sets status to in_progress)
mcp__cas__task action=notes      - Add progress notes (progress, blocker, decision, discovery)
mcp__cas__task action=close      - Complete tasks
mcp__cas__memory action=remember - Store learnings and context
mcp__cas__search action=search   - Find relevant context (filter with doc_type: entry/task/rule/skill)
```

**Human CLI:**
```bash
cas init           # Initialize CAS in a project
cas serve          # Start MCP server
cas config list    # View configuration
cas doctor         # Run diagnostics
cas update         # Self-update
```

## Project Structure

```
cas-cli/           # Rust CLI & MCP server (primary)
crates/            # Workspace crates (cas-core, cas-store, cas-search, etc.)
```

### cas-cli Architecture

| Directory | Purpose |
|-----------|---------|
| `src/types/` | Core data: Entry, Task, Rule, Skill |
| `src/store/` | Storage abstraction (SqliteStore primary) |
| `src/search/` | Full-text search (BM25 via Tantivy) |
| `src/cli/` | Command handlers |
| `src/hooks/` | Claude Code integration |

## Key Patterns

**Store Trait** (`cas-cli/src/store/mod.rs`): All storage ops go through trait abstractions. SqliteStore is primary.

**Rule Auto-Promotion**: Use `mcp__cas__rule action=helpful id=<id>` to promote Draft/Stale rules to Proven, auto-syncs to `.claude/rules/`.

**MCP Server**: `cas serve` exposes all functionality as MCP tools for Claude Code integration.

## Skill Frontmatter Fields (Claude Code 2.1.3+)

CAS skills sync to `.claude/skills/` as SKILL.md files with YAML frontmatter.

**Note:** In Claude Code 2.1.3, skills and commands were unified into a single system.

The following fields are supported:

### Required Fields

| Field | Type | Description |
|-------|------|-------------|
| `name` | string | Skill name (prefixed with `cas-`) |
| `description` | string | 1-2 line description |

### Optional Fields

| Field | Type | Description | Example |
|-------|------|-------------|---------|
| `user-invocable` | bool | Hide from slash menu (model can still invoke) | `false` |
| `argument-hint` | string | Hint shown when invoked (must be quoted) | `"[query]"` |
| `context` | string | Execution context mode | `fork` |
| `agent` | string | Specialized agent to use | `Explore`, `code-reviewer` |
| `allowed-tools` | list | Restrict tools the skill can use | `- Read`<br>`- Grep` |
| `disable-model-invocation` | bool | Block model from invoking (command-only) | `true` |
| `hooks` | object | Skill-scoped hooks (Claude Code 2.1.0+) | See below |

**Note:** `user-invocable: false` only hides the skill from the user's slash menu. The model can still invoke the skill unless `disable-model-invocation: true` is also set.

### Hooks Structure (Claude Code 2.1.0+)

```yaml
hooks:
  PreToolUse:
    - matcher: "Write|Edit"
      hooks:
        - type: command
          command: cas hook PreToolUse
          timeout: 3000
  PostToolUse:
    - matcher: "Write|Edit"
      hooks:
        - type: command
          command: cas hook PostToolUse
          timeout: 3000
  Stop:
    - hooks:
        - type: command
          command: cas hook Stop
```

### Example SKILL.md

```yaml
---
name: cas-deep-search
description: Comprehensive codebase search using forked context
argument-hint: "[query]"
context: fork
agent: Explore
allowed-tools:
  - Read
  - Grep
  - Glob
---

# Deep Search

Instructions for the skill...
```

### Creating Skills

**Via MCP (for agents):**
```
mcp__cas__skill action=create name="My Skill" invokable=true argument_hint="[args]"
```

**Via CLI (for humans):**
```bash
cas skill create "My Skill" --invokable --argument-hint "[args]"
```

Skills are automatically synced to `.claude/skills/` when enabled.

## Hook Configuration (Claude Code 2.1.0+)

### once: true Support

Claude Code 2.1.0 adds `once: true` for hooks that should only execute once per session (even on resume). CAS hooks intentionally do NOT use this because:

- **SessionStart**: Should inject context on every start/resume

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [codingagentsystem/cas](https://github.com/codingagentsystem/cas) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
