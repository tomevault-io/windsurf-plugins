---
trigger: always_on
description: Vibes — a conversational development environment for agent skills, subagents, and documentation.
---

# AGENTS.md

## Project

Vibes — a conversational development environment for agent skills, subagents, and documentation.

This file is the single source of truth for all AI coding assistants.
Chain: `AGENTS.md` (source) ← `CLAUDE.md` (symlink) ← `.github/copilot-instructions.md` (symlink to AGENTS.md)

---

## Fresh Information First

**Do not rely on training data for APIs, SDKs, or framework patterns.**

Always check live documentation before writing code:

| MCP Server | Use For |
|------------|---------|
| **context7** | Library and framework documentation (npm, PyPI, crates, etc.) |
| **microsoft-learn** | Microsoft, Azure, .NET, and M365 documentation |
| **deepwiki** | GitHub repository documentation and wikis |

Skills and agent patterns evolve — verify against current docs, not memory.

---

## Orient First

Run `/primer` at the start of every session before diving into tasks. This analyzes project structure, documentation, key files, and current state — loading essential context for everything that follows.

---

## Core Principles

### 1. Think Before Coding

- Surface assumptions explicitly — don't hide confusion
- Present tradeoffs when multiple approaches exist
- Ask clarifying questions before implementing ambiguous requests
- Plan complex changes before touching code

### 2. Simplicity First

- Write the minimum code that solves the problem
- No speculative features, no "just in case" abstractions
- If you're overcomplicating it, rewrite — don't patch
- Three similar lines beat a premature abstraction

### 3. Surgical Changes

- Touch only what's needed to accomplish the task
- Match existing code style — indentation, naming, patterns
- Don't refactor adjacent code, add docstrings, or "improve" untouched files
- A bug fix doesn't need surrounding cleanup

### 4. Goal-Driven Execution

- Define success criteria as tests before writing implementation (see `.github/docs/tdd-workflow.md`)
- Verify your work — run tests, check output, validate behavior
- Loop until the task is actually done, not just attempted
- If blocked, try a different approach before asking for help

---

## Repository Structure

```
vibes/
├── AGENTS.md                          # Source of truth (this file)
├── CLAUDE.md → AGENTS.md              # Symlink for Claude Code
├── .mcp.json                          # MCP server configuration
│
├── .github/                           # Source of truth for all content
│   ├── skills/                        # Skill source directories
│   │   ├── context7-py/SKILL.md
│   │   ├── context7-sh/SKILL.md
│   │   ├── context7-ps/SKILL.md
│   │   └── ms-learn/SKILL.md
│   ├── agents/                        # Agent definitions
│   │   ├── code-reviewer.md
│   │   └── researcher.md
│   ├── copilot-instructions.md → ../AGENTS.md   # Symlink for Copilot
│   ├── instructions/                  # Copilot scoped instructions
│   │   ├── skill-authoring.instructions.md
│   │   ├── agent-authoring.instructions.md
│   │   ├── documentation.instructions.md
│   │   ├── testing.instructions.md
│   │   └── reference-freshness.instructions.md
│   ├── docs/                          # Platform-agnostic reference docs
│   │   ├── context-engineering.md
│   │   ├── best-practices.md
│   │   └── tdd-workflow.md
│   ├── plugins/
│   └── prompts/
│
├── .claude/                           # Claude Code platform directory
│   ├── skills → ../.github/skills     # Symlink
│   ├── agents → ../.github/agents     # Symlink
│   ├── hooks/                         # Session lifecycle scripts
│   │   ├── backup-transcript.sh       # PreCompact transcript backup
│   │   ├── check-frontmatter.sh       # PostToolUse frontmatter validation
│   │   └── check-symlinks.sh          # PostToolUse symlink integrity
│   ├── settings.json                  # Project hooks config (shareable)
│   ├── references/                    # Knowledge docs (mixed)
│   │   ├── context-engineering.md → ../../.github/docs/context-engineering.md
│   │   ├── best-practices.md → ../../.github/docs/best-practices.md
│   │   ├── tdd-workflow.md → ../../.github/docs/tdd-workflow.md
│   │   ├── skills-guide.md            # Claude-specific
│   │   ├── hooks-guide.md             # Claude-specific
│   │   ├── memory-system.md           # Claude-specific
│   │   └── ...
│   └── rules/                         # Claude Code-specific path rules
│       ├── skill-authoring.md
│       ├── agent-authoring.md
│       ├── documentation.md
│       ├── testing.md
│       └── reference-freshness.md
│
├── .codex/                            # Codex CLI platform directory
│   ├── agents → ../.github/agents     # Symlink
│   ├── skills → ../.github/skills     # Symlink (compatibility alias)
│   ├── config.toml                    # MCP servers, approval policy, sandbox
│   └── rules/                         # Codex command policy (*.rules)
│       └── default.rules
│
├── .agents/                           # Codex-native skills discovery
│   └── skills → ../.github/skills     # Symlink
│
└── skills/                            # Categorized browsing view
    ├── python/
    ├── bash/
    └── powershell/
```

### Architecture

- **`.github/`** is the source of truth for all skills, agents, plugins, and prompts

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jonhill90/vibes](https://github.com/jonhill90/vibes) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->
