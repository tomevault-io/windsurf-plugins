---
trigger: always_on
description: **agentful** is an autonomous product development framework that uses specialized AI agents to build software from a product specification. It coordinates architecture, development, testing, and validation through human-in-the-loop checkpoints, ensuring quality while maintaining 24/7 development velocity.
---

# agentful

**agentful** is an autonomous product development framework that uses specialized AI agents to build software from a product specification. It coordinates architecture, development, testing, and validation through human-in-the-loop checkpoints, ensuring quality while maintaining 24/7 development velocity.

## Installation Options

agentful works with any tech stack (TypeScript, Python, JavaScript, etc.) - the tech stack is auto-detected on first run.

```bash
# Default: Install all components (recommended)
npx @itz4blitz/agentful init

# Minimal: For simple scripts/CLIs
npx @itz4blitz/agentful init --preset=minimal

# Custom: Specify exactly what you want
npx @itz4blitz/agentful init --agents=orchestrator,backend --skills=validation

# View all installation options
npx @itz4blitz/agentful presets
```

## Quick Start

1. Edit `.claude/product/index.md` to define your product requirements
2. Run: `claude`
3. Type: `/agentful-generate`

## Pattern Learning (MCP Server)

Enable cross-session pattern learning so agents compound knowledge over time:

```bash
claude mcp add agentful -- npx -y @itz4blitz/agentful-mcp-server
```

**What this enables:**
- **Reviewer** stores error patterns so the fixer can look up known fixes instantly
- **Fixer** queries known fixes before attempting manual repairs, then stores successful fixes
- **Orchestrator** stores successful implementation patterns after features pass all quality gates

Without MCP: agents start from scratch every session. With MCP: agents compound across sessions.

See [Configuration](#configuration) below for manual setup or other editors.

## Metadata Safety (Critical)

When editing `package.json` (or other repo metadata files), preserve project ownership:

- Never change `repository.url`, `homepage`, `bugs.url`, `name` scope, or `author` to agentful maintainer values unless the project explicitly asks.
- Keep `repository.type` as `"git"` when a `repository` object exists.
- Do not infer repo owner from agentful package names/docs (`@itz4blitz/agentful`); derive owner from the current repository remote instead.
- Prefer structured JSON edits (JSON path / `jq` / `npm pkg`) instead of broad text replace on `"type"` or `"url"`.

## Commands

| Command | Description |
|---------|-------------|
| `/agentful-start` | Begin or resume structured development |
| `/agentful-status` | Check current progress and completion % |
| `/agentful-decide` | Answer pending decisions blocking work |
| `/agentful-validate` | Run all quality checks manually |
| `/agentful-product` | Analyze and improve product specification |

## When to Use What

**Starting fresh?**
→ Run `/agentful-product` to analyze your product spec, then `/agentful-start`

**Existing project?**
→ Run `/agentful-start` directly (auto-detects tech stack)

**Need to check progress?**
→ Run `/agentful-status` to see completion % and current phase

**Validation failures?**
→ The `fixer` agent auto-fixes issues, or run `/agentful-validate` manually

**Agent needs your input?**
→ Check `.agentful/decisions.json` or run `/agentful-decide`

**Unclear requirements?**
→ Run `/agentful-product` in reverse-engineering mode or improve `.claude/product/index.md`

**Want to add features?**
→ Edit `.claude/product/index.md`, then run `/agentful-start` (picks up changes automatically)

## File Structure

**Product Specification** (you edit these):
- `.claude/product/index.md` - Flat structure (all features in one file)
- `.claude/product/domains/` - Hierarchical structure (organized by domain)

**Runtime State** (managed by agentful, gitignored):
- `.agentful/state.json` - Current work phase and progress
- `.agentful/completion.json` - Feature completion % and quality gates
- `.agentful/decisions.json` - Pending and resolved decisions
- `.agentful/conversation-state.json` - Natural language conversation context
- `.agentful/conversation-history.json` - Message history for context tracking
- `.agentful/agent-metrics.json` - Agent lifecycle hooks and metrics
- `.agentful/architecture.json` - Detected tech stack and generated agents
- `.agentful/learnings.json` - Compound engineering retrospectives
- `.agentful/last-validation.json` - Latest validation report from reviewer

**Configuration** (auto-generated, customizable):
- `.claude/agents/` - Specialized agents for your tech stack
- `.claude/commands/` - Slash commands
- `.claude/settings.json` - Hooks and permissions

## Quality Gates

agentful validates code through automated checks:

- **Type checking** - No type errors (TypeScript, Flow, etc.)
- **Linting** - Code follows project style guide
- **Tests** - All tests passing
- **Coverage** - Minimum 80% code coverage
- **Security** - No known vulnerabilities in dependencies
- **Dead code** - No unused exports, files, or dependencies

The `reviewer` agent runs these checks. The `fixer` agent resolves failures.

## Troubleshooting

**"agentful keeps asking me unclear questions"**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [itz4blitz/agentful](https://github.com/itz4blitz/agentful) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
