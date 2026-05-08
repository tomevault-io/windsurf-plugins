---
trigger: always_on
description: > **Contributor guidelines.** Covers rules for developing Mindsystem.
---

# CLAUDE.md

> **Contributor guidelines.** Covers rules for developing Mindsystem.

**Before making any Mindsystem changes, load the `ms-meta` skill** (unless already loaded in this context). It contains architecture knowledge, design principles, and conventions required to make correct decisions.

This document contains rules that affect every output when developing Mindsystem.

## Core Philosophy

Mindsystem is a **meta-prompting system** where every file is both implementation and specification. Files teach Claude how to build software systematically. The system optimizes for:

- **Solo developer + Claude workflow** (no enterprise patterns)
- **Context engineering** (manage Claude's context window deliberately)
- **Plans as prompts** (PLAN.md files are executable, not documents to transform)

Design principles (modularity, main context for collaboration, script+prompt hybrid, user as collaborator) govern all Mindsystem decisions.

---

## Development Context

**All changes happen in this repository.** Never modify user-scope files (`~/.claude/`).

Mindsystem is distributed via `npx mindsystem-cc`. During development, the user runs `npx` which symlinks to this repository. Changes made here are immediately available for testing.

| Location | Purpose |
|----------|---------|
| `agents/` | Subagent definitions (copied to `~/.claude/agents/` on install) |
| `commands/ms/` | Slash commands (copied to `~/.claude/commands/ms/` on install) |
| `mindsystem/` | Workflows, templates, references (copied to `~/.claude/mindsystem/` on install) |
| `scripts/` | CLI tools and scripts (copied to `~/.claude/mindsystem/scripts/` on install) |

**Never write to `~/.claude/` directly.** Always modify files in this repository.

**Testing `ms-tools`:** Run after any modification to `scripts/ms-tools.py`:
```bash
cd scripts && uv run --with pytest --with pyyaml pytest test_ms_tools.py -v
```

**Linear tickets:** References like `MIN-123` are Linear issue IDs. Load the `linear` skill to read, update, or comment on them.

**WARNING:** The `.claude/` directory in the repo root contains tracked project-specific files (settings, custom commands). Do NOT delete it when testing local installations with `npx . --local`. Use a different test directory or restore with `git restore .claude/` if accidentally deleted.

---

## File Structure Conventions

### Slash Commands (`commands/ms/*.md`)

```yaml
---
name: ms:command-name
description: One-line description
argument-hint: "<required>" or "[optional]"
allowed-tools: [Read, Write, Bash, Glob, Grep, AskUserQuestion]
---
```

**Section order:**
1. `<objective>` — What/why/when (always present)
2. `<execution_context>` — @-references to workflows, templates, references
3. `<context>` — Dynamic content: `$ARGUMENTS`, bash output, @file refs
4. `<process>` or `<step>` elements — Implementation steps
5. `<success_criteria>` — Measurable completion checklist

**Commands are thin wrappers.** Delegate detailed logic to workflows.

**Keep command and workflow in sync.** When adding/removing steps in a workflow, update the corresponding command's `<process>` section to match. The command lists steps at a high level; the workflow contains the detailed implementation. Both must reflect the same steps.

### Workflows (`mindsystem/workflows/*.md`)

No YAML frontmatter. Structure varies by workflow.

**Common tags** (not all workflows use all of these):
- `<purpose>` — What this workflow accomplishes
- `<when_to_use>` or `<trigger>` — Decision criteria
- `<required_reading>` — Prerequisite files
- `<process>` — Container for steps
- `<step>` — Individual execution step

Some workflows use domain-specific tags like `<philosophy>`, `<references>`, `<planning_principles>`, `<decimal_phase_numbering>`.

**When using `<step>` elements:**
- `name` attribute: snake_case (e.g., `name="load_project_state"`)
- `priority` attribute: Optional ("first", "second")

**Key principle:** Match the style of the specific workflow you're editing.

### Templates (`mindsystem/templates/*.md`)

Structure varies. Common patterns:
- Most start with `# [Name] Template` header
- Many include a `<template>` block with the actual template content
- Some include examples or guidelines sections

**Placeholder conventions:**
- Square brackets: `[Project Name]`, `[Description]`
- Curly braces: `{phase}-{plan}-PLAN.md`

### References (`mindsystem/references/*.md`)

Typically use outer XML containers related to filename, but structure varies.

Examples:
- `principles.md` → `<principles>...</principles>`
- `plan-format.md` → `<overview>` then `<core_principle>`

Internal organization varies — semantic sub-containers, markdown headers within XML, code examples.

---

## XML Tag Conventions

### Semantic Containers Only

XML tags serve semantic purposes in **commands and workflows**. Use Markdown headers for hierarchy within.

**DO:**
```xml
<objective>
## Primary Goal
Build authentication system

## Success Criteria
- Users can log in
- Sessions persist
</objective>
```

**DON'T:**
```xml
<section name="objective">
  <subsection name="primary-goal">
    <content>Build authentication system</content>
  </subsection>
</section>
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rolandtolnay/mindsystem](https://github.com/rolandtolnay/mindsystem) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
