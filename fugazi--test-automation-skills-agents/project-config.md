---
trigger: always_on
description: This is an AI Agents & Skills repository for test automation.
---

# AGENTS.md

This is an AI Agents & Skills repository for test automation.

The content is **tool-agnostic** (usable with GitHub Copilot, Claude, Cursor, OpenCode, Windsurf, etc.), while the file formats and folder conventions in this repo are primarily optimized for **GitHub Copilot customizations**.

## Repository Structure

```
agents/           # Custom AI agent definitions (*.agent.md)
skills/           # Specialized testing skills (*/SKILL.md)
instructions/     # Guidelines for creating agents/skills (*.instructions.md)
docs/             # Setup guides and documentation
references/       # Shared reference material
```

## Build/Lint/Test Commands

This repository has **no build system** - it is a documentation/knowledge base.

- **No build command** required
- **No linting tools** configured
- **No test runner** present
- Files are Markdown with YAML frontmatter, consumed by GitHub Copilot

## Code Style Guidelines

### File Naming Conventions

| File Type     | Pattern                                  | Example                                 |
| ------------- | ---------------------------------------- | --------------------------------------- |
| Agents        | `lowercase-with-hyphens.agent.md`        | `playwright-test-generator.agent.md`    |
| Instructions  | `lowercase-with-hyphens.instructions.md` | `playwright-typescript.instructions.md` |
| Skills        | `SKILL.md` (inside skill folder)         | `skills/my-skill/SKILL.md`              |
| Skill folders | `lowercase-with-hyphens`                 | `playwright-e2e-testing/`               |

### Frontmatter Requirements

All `.agent.md` and `SKILL.md` files **must** include YAML frontmatter:

```yaml
---
description: "Clear, single-quoted description of purpose (50-150 chars for agents)"
name: "Display Name" # Optional but recommended
tools: ["read", "edit", "search"] # Optional - omit for all tools
model: "Claude Sonnet 4.5" # Strongly recommended
target: "vscode" # Optional: 'vscode' or 'github-copilot'
infer: true # Optional: allow auto-selection (default: true)
---
```

**Skill-specific frontmatter:**

```yaml
---
name: skill-name # Required: lowercase, hyphens, ≤64 chars
description: "WHAT it does, WHEN to use it, KEYWORDS for matching" # Required
license: "Complete terms in LICENSE.txt" # Optional
---
```

### Formatting Standards

- **Line length**: Keep under 120 characters where practical
- **Indentation**: 2 spaces for YAML and Markdown lists
- **Quotes**: Use single quotes for YAML string values
- **Frontmatter markers**: Triple-dash `---` at start and end
- **Markdown headers**: Use `#` for title, `##` for sections, `###` for subsections
- **Bullet lists**: Use `-` (hyphen) with space after
- **Numbered lists**: Use `1.` with space after

### Agent Content Structure

Follow this structure in agent files:

```markdown
# Agent Identity

Clear statement of who the agent is and its primary role.

## Constitution (from TOP)

### MUST DO

- [5-6 rules that are NON-NEGOTIABLE for this agent]

### WON'T DO

- [4-5 rules that this agent must NEVER violate]

## Core Responsibilities

- List specific tasks the agent performs
- Be explicit about scope boundaries

## Approach and Methodology

- How the agent works
- Step-by-step workflow patterns

## Guidelines and Constraints

- What to do/avoid
- Quality standards

## Output Expectations

- Expected format and quality
```

### Constitution Pattern

Agents that generate or modify test code should include a `## Constitution (from TOP)` section defining hard guardrails. The Constitution follows the Test Orchestration Pattern (TOP) and is defined centrally in `agents/qa-orchestrator.agent.md`. Individual agents copy the relevant MUST DO / WON'T DO rules into their own Constitution section so the rules are visible at the agent level.

### Skill Content Structure

```markdown
# Skill Title

Brief overview of capabilities.

## When to Use This Skill

- List of scenarios and triggers

## Prerequisites

Required tools, dependencies, environment setup.

## Step-by-Step Workflows

Numbered steps for common tasks.

## Troubleshooting

| Issue   | Solution |
| ------- | -------- |
| Problem | Fix      |

## References

- [API Reference](./references/api_reference.md)
```

### Directory Organization

**Skills subdirectories:**

```
skills/<skill-name>/
├── SKILL.md              # Required: Main instructions
├── LICENSE.txt           # Recommended: License terms
├── scripts/              # Optional: Executable automation
├── references/           # Optional: Documentation (loaded into context)
├── assets/               # Optional: Static files (used as-is)
└── templates/            # Optional: Starter code (AI modifies)
```

### Tool References

When referencing tools in `tools:` frontmatter:

- Use lowercase aliases: `read`, `edit`, `search`, `execute`, `agent`, `web`
- For MCP servers: `playwright/*`, `github/*`
- For specific tools: `playwright/navigate`, `github/get_file_contents`

### Variable Usage

Use `${variableName}` syntax for dynamic values in prompts:

```markdown
## Dynamic Parameters

- **projectName**: ${projectName}
- **basePath**: ${basePath}
```

### Best Practices

1. **Be specific and direct**: Use imperative mood ("Analyze", "Generate")

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fugazi/test-automation-skills-agents](https://github.com/fugazi/test-automation-skills-agents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
