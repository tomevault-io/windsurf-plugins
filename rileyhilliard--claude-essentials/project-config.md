---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a unified Claude Code plugin (`ce`) that provides development workflows, reusable skills, and specialized agents, all under a consistent `ce` namespace.

**The ce plugin provides:**

- **14 Commands** - Development workflows (test, explain, debug, optimize, refactor, review, commit, deps, fix-issue, pr, document, plan, execute, init)
- **18 Skills** - Reusable patterns for testing, debugging, refactoring, architecture, and planning
- **3 Agents** - Expert AI personas (code-reviewer, haiku, log-reader)
- **Session Hooks** - Automatic project configuration on startup
- **Reference Templates** - ADR, PRD, and technical design templates

**Namespace conventions:**

- Commands: `/ce:test`, `/ce:explain`, `/ce:commit`, `/ce:plan`, etc.
- Skills: `@skills/ce:writing-tests`, `@skills/ce:systematic-debugging`, `@skills/ce:architecting-systems`, etc.
- Agents: `@ce:code-reviewer`, `@ce:haiku`, `@ce:log-reader`

The `ce:` prefix is automatically added by Claude Code based on the plugin name. Files and YAML frontmatter use simple names without the prefix.

## Plugin Architecture

### Directory Structure

The ce plugin lives in `plugins/ce/` with this structure:

```
plugins/ce/
├── .claude-plugin/
│   └── plugin.json          # Plugin metadata (name: "ce", description, version, author, license)
├── commands/                 # 14 slash commands
│   ├── test.md              # Accessed as /ce:test
│   ├── explain.md           # Accessed as /ce:explain
│   ├── debug.md             # Accessed as /ce:debug
│   ├── optimize.md          # Accessed as /ce:optimize
│   ├── refactor.md          # Accessed as /ce:refactor
│   ├── review.md            # Accessed as /ce:review
│   ├── commit.md            # Accessed as /ce:commit
│   ├── deps.md              # Accessed as /ce:deps
│   ├── fix-issue.md         # Accessed as /ce:fix-issue
│   ├── pr.md                # Accessed as /ce:pr
│   ├── document.md          # Accessed as /ce:document
│   ├── plan.md              # Accessed as /ce:plan
│   ├── execute.md           # Accessed as /ce:execute
│   └── init.md              # Accessed as /ce:init
├── skills/                   # 18 skills
│   ├── writing-tests/       # Accessed as @skills/ce:writing-tests
│   │   └── SKILL.md         # name: writing-tests (no ce: prefix in file)
│   ├── architecting-systems/    # Accessed as @skills/ce:architecting-systems
│   │   └── SKILL.md             # System architecture and technical docs
│   └── ...                  # Other skills follow same pattern
├── agents/                   # 3 agents
│   ├── code-reviewer.md     # Accessed as @ce:code-reviewer
│   ├── haiku.md             # Accessed as @ce:haiku
│   └── log-reader.md        # Accessed as @ce:log-reader
└── hooks/                    # Session hooks
    ├── hooks.json           # Hook configuration
    ├── session-start.sh     # Session startup hook
    └── notify.sh            # Cross-platform notification hook
```

**Key principle**: Files and frontmatter use simple names (e.g., `architecting-systems`, `writing-tests`). Claude Code automatically adds the `ce:` namespace prefix based on the plugin name.

### Plugin Metadata

Every plugin requires `.claude-plugin/plugin.json`:

```json
{
  "name": "plugin-name",
  "description": "What the plugin does",
  "version": "1.0.0",
  "author": {
    "name": "Riley Hilliard"
  },
  "license": "MIT"
}
```

### Marketplace Configuration

The marketplace configuration in `.claude-plugin/marketplace.json` defines the ce plugin.

**Valid plugin fields in marketplace.json:**

- **Metadata**: `name`, `version`, `description`, `author`, `homepage`, `repository`, `license`, `keywords`
- **Component paths**: `commands`, `agents`, `skills`, `hooks`, `mcpServers`
- **Marketplace-specific**: `source`, `category`, `tags`, `strict`

**Important**:

- The `hooks` field, if declared in marketplace.json, must be an **inline object** (not a string path). Alternatively, omit it and let Claude Code auto-discover the `hooks/hooks.json` file in the plugin directory.
- The `references` field is NOT supported in marketplace.json.
- File paths in `commands`, `skills`, and `agents` arrays must NOT include namespace prefixes (e.g., use `./skills/writing-tests` not `./skills/ce:writing-tests`).

### Command Format

Commands in `commands/*.md` use YAML frontmatter:

```markdown
---
description: What the command does
argument-hint: "[optional-args]"
model: sonnet
allowed-tools: Bash, Read, Grep
---

Command instructions here.
Use $ARGUMENTS for user-provided arguments.
```

### Skill Format

Skills in `skills/<skill-name>/SKILL.md` use YAML frontmatter:

```markdown
---
name: skill-name
description: What the skill does and when to use it (max 1024 chars)
---

# Skill Title

Skill instructions using imperative language.
```

**Skill naming conventions:**

- Use gerund form: `writing-tests`, `debugging-code`, `creating-skills`
- Lowercase with hyphens only
- No reserved words ("anthropic", "claude")
- Maximum 64 characters

**Description guidelines:**

- Third person only (injected into system prompt)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rileyhilliard) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
