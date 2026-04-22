---
trigger: always_on
description: > A meta-project for setting up new Claude Code projects with optimal CLAUDE.md files, agent configurations, and skill definitions.
---

# Claude Code Project Scaffolder

> A meta-project for setting up new Claude Code projects with optimal CLAUDE.md files, agent configurations, and skill definitions.

## IMPORTANT: Reference Documentation

Before scaffolding projects or generating CLAUDE.md files, READ these authoritative sources to ensure you're using the latest best practices:

### Required Reading (fetch these URLs)
1. **Claude Code Best Practices**: https://www.anthropic.com/engineering/claude-code-best-practices
2. **Building Effective Agents**: https://www.anthropic.com/engineering/building-effective-agents
3. **Agent Skills Guide**: https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills
4. **Claude Agent SDK**: https://www.anthropic.com/engineering/building-agents-with-the-claude-agent-sdk
5. **Subagents Documentation**: https://code.claude.com/docs/en/sub-agents
6. **CLAUDE.md Files Guide**: https://www.claude.com/blog/using-claude-md-files

When working on a scaffolding task, fetch the relevant URLs above to ensure recommendations are current. The Anthropic engineering blog is regularly updated with new patterns and practices.

## Project Purpose

This project helps developers quickly scaffold new Claude Code projects with:
- Well-structured CLAUDE.md files tailored to their stack
- Custom slash commands for their workflows
- Subagent configurations for specialized tasks
- Agent skills for domain-specific capabilities
- MCP server configurations
- Best-practice project structures

## Core Principles

### 1. Progressive Disclosure
When generating CLAUDE.md files, follow the principle of **progressive disclosure**:
- Include only universally-applicable instructions in CLAUDE.md
- Reference external documentation for context-specific details
- Use slash commands for specialized workflows
- Don't overload the context window with rarely-used instructions

### 2. Less Is More
A good CLAUDE.md should be:
- **Concise**: Every line should be universally applicable
- **Actionable**: Tell Claude HOW to do things, not just WHAT to do
- **Verifiable**: Include commands Claude can run to validate its work

### 3. Never Send an LLM to Do a Linter's Job
- Don't include code style guidelines in CLAUDE.md
- Instead, reference linter/formatter configs
- Use hooks to run formatters automatically

## Commands

```bash
# Scaffolding commands
/project:scaffold         # Interactive project setup wizard
/project:scaffold-react   # Quick React/TypeScript project
/project:scaffold-python  # Quick Python project
/project:scaffold-api     # Quick API project (FastAPI/Express)

# CLAUDE.md commands  
/project:generate-claude-md    # Generate CLAUDE.md from project analysis
/project:optimize-claude-md    # Optimize existing CLAUDE.md

# Subagent commands
/project:create-subagent  # Create a specialized subagent
/agents                   # Manage subagents (built-in command)

# Skills commands
/project:create-skill     # Create a new agent skill
/project:list-skills      # List available skill templates

# Analysis commands
/project:analyze          # Analyze codebase and suggest improvements
```

## Project Structure

```
project-scaffolder/
├── CLAUDE.md                    # This file
├── .claude/
│   ├── commands/               # This project's slash commands
│   │   ├── scaffold.md
│   │   ├── scaffold-react.md
│   │   ├── scaffold-python.md
│   │   ├── generate-claude-md.md
│   │   ├── create-skill.md
│   │   └── create-subagent.md
│   └── agents/                 # Subagent definitions
│       ├── code-reviewer.md
│       ├── project-analyzer.md
│       └── test-runner.md
├── templates/                   # Project templates
│   ├── claude-md/              # CLAUDE.md templates by project type
│   │   ├── react-typescript.md
│   │   ├── python-api.md
│   │   ├── fullstack.md
│   │   └── minimal.md
│   ├── commands/               # Slash command templates
│   │   ├── review.md
│   │   ├── test-and-commit.md
│   │   └── fix-issue.md
│   ├── agents/                 # Subagent templates
│   │   ├── code-reviewer.md
│   │   ├── debugger.md
│   │   └── researcher.md
│   └── skills/                 # Skill templates
│       └── SKILL.template.md
├── skills/                      # This project's skills
│   └── project-setup/
│       └── SKILL.md
└── docs/
    ├── best-practices.md       # CLAUDE.md best practices
    ├── agent-patterns.md       # Common agent patterns
    ├── skill-guide.md          # How to write skills
    └── subagent-guide.md       # How to create subagents
```

## Subagents

Subagents are specialized AI assistants that Claude Code can delegate tasks to. Each subagent:
- Has a specific purpose and expertise area
- Uses its own context window (preserves main conversation context)
- Can be configured with specific tools
- Has a custom system prompt guiding behavior

### Built-in Subagents
- **Explore**: Fast, read-only codebase search (uses Haiku)
- **Plan**: Research for plan mode
- **General-purpose**: Complex multi-step tasks

### Creating Custom Subagents
Place `.md` files in `.claude/agents/` with this format:

```markdown
---
name: subagent-name
description: When to use this subagent (be specific!)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fltman/project-scaffolder](https://github.com/fltman/project-scaffolder) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
