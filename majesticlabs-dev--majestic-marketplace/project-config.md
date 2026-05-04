---
trigger: always_on
description: Claude Code plugin marketplace. **Work in `plugins/*/` only.**
---

# Majestic Marketplace

Claude Code plugin marketplace. **Work in `plugins/*/` only.**

**Quick Nav:** [Forbidden](#forbidden) · [Structure](#structure) · [Dependencies](#dependencies) · [Docs](#documentation) · [Search](#codebase-search-qmd) · [Rules](#key-rules) · [Release](#plugin-release-checklist)

## FORBIDDEN

NEVER modify `~/.claude/`. All plugin work goes in `majestic-marketplace/plugins/`.

## Git Merge Operations

- NEVER merge branches to master without explicit user approval
- Always ask: "Ready to merge to master?" or create a PR for review
- Merging to production is irreversible - requires conscious decision

## Structure

```
plugins/{engineer,rails,python,react,marketing,sales,company,llm,tools,devops,experts}/
```

**Wiki repo**: `../majestic-marketplace.wiki/` (separate git repo, requires separate commit/push)

## Dependencies

| Plugin | Can Reference |
|--------|--------------|
| `engineer` | rails, python, react, llm, tools |
| `rails`, `python`, `react` | engineer |
| `marketing`, `sales`, `company`, `llm`, `tools`, `devops`, `experts` | (none) |

## Documentation

- [Naming](docs/plugin-architecture/NAMING-CONVENTIONS.md)
- [Operations](docs/plugin-architecture/PLUGIN-OPERATIONS.md)
- [Schemas](docs/plugin-architecture/JSON-SCHEMAS.md)
- [Config](docs/plugin-architecture/CONFIG-SYSTEM.md)

## Codebase Search (qmd)

Use `qmd` to search indexed plugin docs, skills, agents, and architecture files (622 markdown files). Always add `--json` for structured output.

| Need | Command |
|------|---------|
| Exact keyword | `qmd search "blueprint workflow" --json` |
| Conceptual/semantic | `qmd vsearch "how does task tracking work" --json` |
| Complex question | `qmd query "agent vs skill pattern" --json` |
| Read a doc | `qmd get path/to/file.md --json` |
| Batch read | `qmd multi-get "plugins/engineer/agents/**/*.md" --json` |

- Collection: `majestic-marketplace` (restrict with `-c majestic-marketplace`)
- Prefer `search` for known terms, `vsearch` for fuzzy/conceptual, `query` for best quality
- Use before Glob/Grep when searching across plugin documentation or skill content
- Re-index after major changes: `qmd collection remove majestic-marketplace && qmd collection add . --name majestic-marketplace && qmd embed`

## Config Access

```
VARIABLE = config_read("field", "default")
```

`config_read(field, default)` is pseudocode for invoking the `config-reader` skill (reads `.agents.yml` with `.agents.local.yml` overrides, supports dot notation for nested fields).

## Key Rules

### Limits
- Skills: <500 lines
- Agents: <300 lines

### Step Numbering Conventions

**Apply on first draft, not after correction.**

| Pattern | When to Use | Example |
|---------|-------------|---------|
| `1, 2, 3...` | Main sequential steps | Step 1, Step 2, Step 3 |
| `1.1, 1.2` | Substeps or alternatives under a parent | Step 1.1, Step 1.2, Step 1.3 |

**Rules:**
- Start numbering at 1, never 0
- Prefer adding new main steps over decimals (renumber if needed)
- Never use: `0A`, `8.5`, `Step 0`, letter suffixes, `2.5`

### File Locations
- Skill references → `skills/*/references/`
- Skill assets → `skills/*/assets/`
- Skill scripts → `skills/*/scripts/`
- Agent resources → `agents/**/resources/{agent-name}/`
- Command resources → `commands/**/resources/{command-name}/`
- Resources referenced via relative paths from the markdown file
- No `.md` files in `commands/` (they become executable)
- Templates in command resources must use `.txt` or `.yml` extensions

**Resource file patterns:**
- One primary concern per file (don't mix unrelated templates)
- Keep shared content in parent folder, reference via relative path
  - Example: email-nurture references `../email-sequences/subject-formulas.md`
- Split files if they serve different purposes:
  - Good: email-structure.md + re-engagement-copy.md
  - Bad: email-copy-template.md (mixing everything)
- Avoid duplication: If two agents need same resource, one file in shared location

### Behaviors
- Skills = knowledge (Claude MAY follow)
- Hooks = enforcement (FORCES behavior)
- Agents do autonomous work, not just advice
- `name:` in frontmatter overrides path-based naming

**Skill vs Agent:**

| Type | Execution | Tools | Convert When |
|------|-----------|-------|--------------|
| Skill | Inline (main conversation) | Optional `allowed-tools` | N/A — default for guidance |
| Agent | Subprocess (Task tool) | Own context | Autonomous multi-step workflow |

Wrong: "Has tools → must be agent" · Right: "Does autonomous workflow → should be agent"

### Agent/Skill Pairing Pattern
When an agent invokes a skill:
- AGENT responsibility: Workflow steps, validation, error handling, delivery
  - Gather context → Validate inputs → Analyze situation → Invoke skill → Validate output
- SKILL responsibility: Templates, frameworks, patterns
  - Receives structured context from agent
  - Does the actual work (writes, builds, creates)
- RULE: Remove duplication — if agent contains the template, don't duplicate in skill

### Command Naming
- Bare names by default: `name: command-name` → invoked as `/command-name`
- Choose self-descriptive verbs (`/commit`, `/migrate`, `/triage-prs`)
- Add prefix only when bare loses meaning (workflow-starters like `start`, `new`):

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [majesticlabs-dev/majestic-marketplace](https://github.com/majesticlabs-dev/majestic-marketplace) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
