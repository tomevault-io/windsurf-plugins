---
trigger: always_on
description: The **Copilot Playbook** is a centralized collection of custom agents, instructions, and skills for GitHub Copilot. It provides standardized guidance for code review, workflow automation, development practices, and quality standards, designed to be reusable across multiple C# projects.
---

# Copilot Playbook — Claude Code Guide

## Project Overview

The **Copilot Playbook** is a centralized collection of custom agents, instructions, and skills for GitHub Copilot. It provides standardized guidance for code review, workflow automation, development practices, and quality standards, designed to be reusable across multiple C# projects.

## Directory Structure

```txt
ai-playbook/
├── plugins/                        # All plugins (each with .claude-plugin/plugin.json + skills/)
│   ├── bruno/
│   │   ├── .claude-plugin/plugin.json
│   │   └── skills/
│   │       ├── bruno-e2e/SKILL.md
│   │       └── bruno-generator/SKILL.md
│   ├── code-auditor/
│   │   ├── .claude-plugin/plugin.json
│   │   ├── commands/audit.md       # /audit [pr|full]
│   │   ├── agents/auditor.md       # autonomous read-only audit agent
│   │   └── skills/
│   │       ├── audit-architecture/SKILL.md
│   │       ├── audit-security/SKILL.md
│   │       └── audit-quality/SKILL.md
│   ├── dotnet/
│   │   ├── .claude-plugin/plugin.json
│   │   └── skills/
│   │       ├── csharp-conventions/SKILL.md
│   │       └── dotnet-check/SKILL.md
│   ├── flutter/
│   │   ├── .claude-plugin/plugin.json
│   │   └── skills/
│   │       ├── flutter-architecture/SKILL.md
│   │       ├── flutter-orient-ui/SKILL.md
│   │       └── flutter-style/SKILL.md
│   └── git-workflow/
│       ├── .claude-plugin/plugin.json
│       └── skills/
│           ├── git-branch-naming/SKILL.md
│           ├── git-conventional-commit/SKILL.md
│           └── git-pull-request-formatting/SKILL.md
│
├── .claude-plugin/
│   └── marketplace.json            # Claude Code marketplace (5 plugins, all via source: ./plugins/{name})
│
├── .github/
│   └── workflows/                  # CI/CD pipelines
│
├── apm.yml                         # Agent Package Manager dependencies
└── mise.toml                       # Tool version management
```

## Skill Framework

### What is a Skill?

A skill is a reusable, self-contained capability that can be invoked from Claude Code or GitHub Copilot. Each skill is defined as a `SKILL.md` file containing:

- **YAML Frontmatter**: Metadata about the skill
- **Content**: Step-by-step instructions or guidelines

### SKILL.md Format

```yaml
---
name: skill-identifier
description: Brief description of what the skill does and when to use it
---
# Step-by-step instructions or guidelines here
```

### Available Skills

| Skill                           | Purpose                                     | Category        |
| ------------------------------- | ------------------------------------------- | --------------- |
| **audit-architecture**          | Tech-agnostic Clean Architecture audit      | Quality Review  |
| **audit-security**              | Secret leak & vulnerability detection       | Quality Review  |
| **audit-quality**               | Demanding code quality audit                | Quality Review  |
| **bruno-e2e**                   | Run Bruno API tests interactively           | Testing         |
| **bruno-generator**             | Generate Bruno .bru test files              | Code Generation |
| **csharp-conventions**          | C# and .NET coding conventions (auto)       | Conventions     |
| **dotnet-check**                | Build C# project & run unit tests           | Build/Test      |
| **flutter-architecture**        | Flutter feature-first architecture (auto)   | Conventions     |
| **flutter-orient-ui**           | Orient UI component usage in Flutter (auto) | Conventions     |
| **flutter-style**               | Flutter styling for Apple compliance (auto) | Conventions     |
| **git-branch-naming**           | Git branch naming convention (auto)         | Documentation   |
| **git-conventional-commit**     | Conventional Commits format guide           | Documentation   |
| **git-pull-request-formatting** | PR title and description format             | Documentation   |

### Claude Code Marketplace Plugins

Five plugins are declared in `.claude-plugin/marketplace.json`. Every plugin lives under `plugins/{name}/` with its own `plugin.json` and `skills/` directory — skills are auto-discovered by convention, no explicit `skills` paths needed in `marketplace.json`:

| Plugin           | Source                    | Contents                                                                             |
| ---------------- | ------------------------- | ------------------------------------------------------------------------------------ |
| **bruno**        | `./plugins/bruno`         | bruno-e2e, bruno-generator                                                           |
| **code-auditor** | `./plugins/code-auditor`  | `/audit` command, `auditor` agent, audit-architecture, audit-security, audit-quality |
| **dotnet**       | `./plugins/dotnet`        | csharp-conventions, dotnet-check                                                     |
| **flutter**      | `./plugins/flutter`       | flutter-architecture, flutter-orient-ui, flutter-style                               |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jterral/ai-playbook](https://github.com/jterral/ai-playbook) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
