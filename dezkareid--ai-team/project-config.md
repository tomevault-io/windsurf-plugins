---
trigger: always_on
description: This project is a collection of AI utils like agents/commands/skills and more that are intended to be used together to use AI tools more effectively.
---

# Agent Instructions: AI Team Project

This project is a collection of AI utils like agents/commands/skills and more that are intended to be used together to use AI tools more effectively.

## Overview

Support different AI agents/commands/skills formats:

### Command File Formats

#### Markdown Format

Used by: Claude Code, Cursor, opencode, Windsurf, Amazon Q Developer, Amp, SHAI, IBM Bob

**Standard format:**

Used by Claude Code

```markdown
---
description: "Command description"
---

Command content with {SCRIPT} and $ARGUMENTS placeholders.
```

#### TOML Format

Used by: Gemini

```toml
description = "Command description"

prompt = """
Command content with {SCRIPT} and {{args}} placeholders.
"""
```

### Organization

- **`commands/`**: Source command files in TOML format, primarily used by Gemini CLI.
- **`skills/`**: Source skill files (SKILL.md), providing specialized knowledge and instructions.
- **`plugins/`**: Claude Code plugins. Each subdirectory represents a plugin and contains exported versions of commands and skills in Markdown format.
- **`.claude-plugin/`**: Metadata and configuration for Claude Code. The root folder contains the `marketplace.json`, while plugin folders contain their respective `plugin.json`.

### AI Team MCP

The `ai-team` MCP server provides authoritative business and architectural context.

#### Available Tools

| Tool | Description |
|---|---|
| `get_enterprise_context` | Retrieves the enterprise mission, strategic goals, and core architecture characteristics. |
| `get_company_outcomes` | Retrieves the high-level business outcomes and key results. |
| `get_architecture_principles` | Retrieves the technology-agnostic architecture principles. |
| `search_product` | Dynamic tool to search for specific product characteristics. |

#### Context Folder Structure

```text
context/
├── enterprise.md              # Shared mission, goals, and core characteristics
├── outcomes.md                # Shared strategic outcomes (OKRs)
├── architecture-principles.md # Fundamental philosophy and standards
└── products/                  # Product-specific characteristics
    ├── personal-website.md
    ├── collecstory.md
    └── default.md             # Fallback for undocumented products
```

#### Project Configuration (`.agent-structurerc`)

File `.agent-structurerc` is used to configure the project structure.

```json
{
  "claude-plugins": {
    "npm-tools": {
      "name": "npm-tools",
      "version": "0.0.1",
      "description": "Tools for working with npm"
    },
    "design-system": {
      "name": "design-system",
      "version": "0.0.1",
      "description": "Authoritative design system context and tools"
    },
    "company-context": {
      "name": "company-context",
      "version": "0.0.3",
      "description": "Authoritative company context and tools"
    },
    "web-quality": {
      "name": "web-quality",
      "version": "0.0.1",
      "description": "Skills for auditing and optimizing web quality (Performance, Accessibility, SEO, Core Web Vitals, Best Practices)"
    },
    "frontend-tools": {
      "name": "frontend-tools",
      "version": "0.0.1",
      "description": "Expert procedural guidance for frontend development (React, Next.js, etc.)"
    },
    "database-tools": {
      "name": "database-tools",
      "version": "0.0.1",
      "description": "Expert procedural guidance for database optimization and best practices"
    }
  },
  "commands": [
    {
      "name": "npm-package-setup",
      "source": "npm/package-setup.toml",
      "claude-plugin": "npm-tools"
    },
    {
      "name": "npm-publish",
      "source": "npm/publish-setup.toml",
      "claude-plugin": "npm-tools"
    }
  ],
  "skills": [
    {
      "name": "design-tokens",
      "source": "skills/design-tokens/SKILL.md",
      "claude-plugin": "design-system"
    },
    {
      "name": "performance",
      "source": "skills/performance/SKILL.md",
      "claude-plugin": "web-quality"
    },
    {
      "name": "accessibility",
      "source": "skills/accessibility/SKILL.md",
      "claude-plugin": "web-quality"
    },
    {
      "name": "seo",
      "source": "skills/seo/SKILL.md",
      "claude-plugin": "web-quality"
    },
    {
      "name": "core-web-vitals",
      "source": "skills/core-web-vitals/SKILL.md",
      "claude-plugin": "web-quality"
    },
    {
      "name": "best-practices",
      "source": "skills/best-practices/SKILL.md",
      "claude-plugin": "web-quality"
    },
    {
      "name": "web-quality-audit",
      "source": "skills/web-quality-audit/SKILL.md",
      "claude-plugin": "web-quality"
    },
    {
      "name": "react-best-practices",
      "source": "skills/react-best-practices/SKILL.md",
      "claude-plugin": "frontend-tools"
    },
    {
      "name": "next-best-practices",
      "source": "skills/next-best-practices/SKILL.md",
      "claude-plugin": "frontend-tools"
    },
    {
      "name": "supabase-postgres-best-practices",
      "source": "skills/supabase-postgres-best-practices/SKILL.md",
      "claude-plugin": "database-tools"
    }
  ],
  "mainMcp": {
    "claude-plugin": "company-context",
    "version": "1.4.6",
    "package": "@dezkareid/ai-team",

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dezkareid/ai-team](https://github.com/dezkareid/ai-team) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
