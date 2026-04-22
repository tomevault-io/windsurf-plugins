---
trigger: always_on
description: This file provides guidance to AI coding agents (Claude Code, Cursor, Copilot, etc.) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to AI coding agents (Claude Code, Cursor, Copilot, etc.) when working with code in this repository.

## Repository Overview

A collection of skills for AI agents working with ClickHouse databases and chdb (in-process ClickHouse for Python). Skills are packaged instructions and guidelines that extend agent capabilities for database design, query optimization, operational best practices, and in-process data analytics.

## Repository Structure

```
agent-skills/
├── skills/
│   ├── clickhouse-best-practices/   # ClickHouse optimization guidelines
│   │   ├── SKILL.md                 # Skill definition (overview)
│   │   ├── AGENTS.md                # Full compiled guide (generated)
│   │   ├── metadata.json            # Version, organization, abstract
│   │   ├── README.md                # Maintainer guide
│   │   └── rules/                   # Individual rule files
│   │       ├── _sections.md         # Section metadata
│   │       ├── _template.md         # Template for new rules
│   │       └── *.md                 # Rule files (e.g., query-use-prewhere.md)
│   ├── chdb-datastore/              # chdb pandas-compatible DataStore API
│   │   ├── SKILL.md                 # Skill definition and quick-start
│   │   ├── metadata.json            # Version, organization, abstract
│   │   ├── README.md                # Maintainer guide
│   │   ├── references/              # API reference docs
│   │   │   ├── api-reference.md     # DataStore method signatures
│   │   │   └── connectors.md        # All data source connection methods
│   │   ├── examples/
│   │   │   └── examples.md          # Runnable examples
│   │   └── scripts/
│   │       └── verify_install.py    # Environment verification
│   └── chdb-sql/                    # chdb SQL API
│       ├── SKILL.md                 # Skill definition and quick-start
│       ├── metadata.json            # Version, organization, abstract
│       ├── README.md                # Maintainer guide
│       ├── references/              # SQL reference docs
│       │   ├── api-reference.md     # query/Session/connect signatures
│       │   ├── table-functions.md   # ClickHouse table functions
│       │   └── sql-functions.md     # Commonly used SQL functions
│       ├── examples/
│       │   └── examples.md          # Runnable examples
│       └── scripts/
│           └── verify_install.py    # Environment verification
├── packages/
│   └── clickhouse-best-practices-build/  # Build tooling
│       ├── package.json             # Bun scripts
│       ├── tsconfig.json            # TypeScript config
│       └── src/
│           ├── config.ts            # Path configuration
│           ├── types.ts             # Type definitions
│           ├── parser.ts            # Markdown parser
│           ├── build.ts             # Build script
│           ├── validate.ts          # Rule validator
│           ├── validate-sql.ts      # SQL syntax validator
│           └── check-links.ts       # Internal link checker
└── .github/
    └── workflows/
        └── clickhouse-best-practices-ci.yml  # CI workflow
```

## Creating a New Skill

### Directory Structure

```
skills/
  {skill-name}/           # kebab-case directory name
    SKILL.md              # Required: skill definition
    AGENTS.md             # Generated: full compiled guide
    metadata.json         # Required: version, organization, abstract
    README.md             # Required: maintainer guide
    rules/                # Required: rule files
      _sections.md        # Section metadata
      _template.md        # Template for new rules
      *.md                # Individual rules
```

### Naming Conventions

- **Skill directory**: `kebab-case` (e.g., `clickhouse-best-practices`)
- **SKILL.md**: Always uppercase, always this exact filename
- **Rule files**: `{section-prefix}-{descriptive-name}.md` (e.g., `query-use-prewhere.md`)
- **Section prefixes**: Match the section IDs defined in `_sections.md`

### SKILL.md Format

```markdown
---
name: {skill-name}
description: {One sentence describing when to use this skill. Include trigger phrases.}
license: MIT
metadata:
  author: {organization}
  version: "{version}"
---

# {Skill Title}

{Brief description of what the skill does.}

## When to Apply

Reference these guidelines when:
- {Use case 1}
- {Use case 2}

## Rule Categories by Priority

| Priority | Category | Impact | Prefix |
|----------|----------|--------|--------|
| 1 | {Category} | {Impact} | `{prefix}-` |

## Quick Reference

{Brief overview of each category and key rules}

## How to Use

{Instructions on reading individual rule files}

## Full Compiled Document

For the complete guide with all rules expanded: `AGENTS.md`
```

### Rule File Format

Use the template in `rules/_template.md`. Each rule file must have:

1. **YAML frontmatter**:
   ```yaml
   ---
   title: Rule Title
   impact: CRITICAL | HIGH | MEDIUM-HIGH | MEDIUM | LOW-MEDIUM | LOW
   impactDescription: Optional (e.g., "10-100× query speedup")
   tags: skill, category, specific-tags
   ---
   ```

2. **Rule structure**:
   - Brief explanation of why it matters
   - **Incorrect:** code example showing the anti-pattern

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ClickHouse/agent-skills](https://github.com/ClickHouse/agent-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
