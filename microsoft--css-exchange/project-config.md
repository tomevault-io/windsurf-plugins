---
trigger: always_on
description: NEVER commit, push, or create PRs without explicit user instruction.
---

# Copilot Instructions for CSS-Exchange

## Behavioral Rules

NEVER commit, push, or create PRs without explicit user instruction.
NEVER auto-fix issues found during code review. Report findings only.
NEVER present code as complete without running `Invoke-CodeFormatterOnFiles` and `SpellCheck.ps1` first.
ALWAYS report findings and wait for user decision before modifying the repository.
ALWAYS commit only the specific files the user requests. No extra files.
ALWAYS use named parameters instead of positional parameters.
ALWAYS ensure .ps1 files have UTF-8 BOM encoding.
When counting code patterns (function calls, occurrences), ALWAYS exclude comment lines from the count.

## Code Quality Standards

All code must pass these checks before being presented as complete:

```powershell
. .build/Invoke-CodeFormatterOnFiles.ps1
Invoke-CodeFormatterOnFiles -FilePaths @("<your changed files>") -Save   # Formatting, BOM, PSScriptAnalyzer
.build/SpellCheck.ps1                                                    # Spelling (use PascalCase/camelCase)
```

## Available Agents and Skills

Specialized agents are available in `.github/agents/` for specific tasks:

- **code-review** — Runs the quality pipeline and analyzes code for issues. Reports findings only, cannot modify files. Use: `"Use the code-review agent to review this PR"`
- **code-creator** — Creates and modifies PowerShell scripts following repo standards. Validates with `Invoke-CodeFormatterOnFiles` and `SpellCheck.ps1`. Use: `"Use the code-creator agent to create a new function"`
- **pester-test** — Creates Pester tests following repo patterns and performance rules. Use: `"Use the pester-test agent to write tests for FunctionName"`

Skills are available in `.github/skills/` and are automatically loaded when relevant:

- **dependency-analysis** — Maps cascading impact of changes using build system XML. Auto-loads when working with shared functions or assessing change impact. Can also be invoked explicitly: `"Use the /dependency-analysis skill to analyze impact of changing FunctionName"`

## Personal Overrides

Developers can create personal agents, skills, and instructions that override or extend the repo-level defaults without affecting other contributors. These directories are gitignored:

| Directory | Purpose | Behavior |
|-----------|---------|----------|
| `.copilot/copilot-instructions.md` | Personal instructions | **Merges** with repo instructions (additive — safe) |
| `.copilot/agents/` | Personal agent overrides | **Replaces** repo agent entirely (use with caution) |
| `.copilot/skills/` | Personal skills | Available only to you |
| `.claude/` | Claude Code personal configuration | — |
| `.agents/` | Alternative personal agents/skills location | — |

### Important: Agent Overrides Replace, They Don't Merge

If you create `.copilot/agents/code-review.md`, it **completely replaces** `.github/agents/code-review.agent.md` — it does not merge. This means:
- You must maintain the full agent file, not just your changes
- You won't receive repo-level updates to that agent until you delete your override
- Stale overrides silently mask improvements made by other contributors

### Recommended Workflow for Iterating on Rules

1. **Discover** a needed change while working → store it as a Copilot memory
2. **Test** additive changes via `.copilot/copilot-instructions.md` (safe — merges with repo)
3. **Only** use `.copilot/agents/` override when testing a full agent rewrite, and delete the override once the change is promoted
4. **Batch** validated changes into a single PR to `.github/` periodically
5. **Clean up** any memories that were promoted to files

## Repository Overview

CSS-Exchange is a multi-script repository where:
- **Main scripts** live in domain folders (Admin, Calendar, Databases, Diagnostics, Hybrid, M365, Outlook, etc.)
- **Shared utilities** are in the `Shared/` folder and are dot-sourced into scripts
- **Scripts are built** into single `.ps1` files in the `dist/` folder via the build system
- **All scripts follow PowerShell best practices** enforced by PSScriptAnalyzer and custom formatting tools

## Before Any Work: Understanding Script Context

Before writing tests, reviewing code, or making any changes to a script:

1. Read the full script to understand its purpose, logic, functions, parameters, and dependencies.
2. Use `.build/Build.ps1` to generate dependency XML files (`dist/dependencyHashtable.xml` and `dist/dependentHashtable.xml`). These are the authoritative source for all script relationships.
3. Map the cascading impact: changes to Shared functions affect direct callers AND their callers. The dependency-analysis skill auto-loads when relevant, or use `.github/skills/dependency-analysis/Get-DependencyCascade.ps1` directly.

## Build, Test, and Code Quality Commands

All commands require **PowerShell 7+** and should be run from the repository root.

### Building

```powershell
.build/Build.ps1
```

- Combines multi-file scripts into single-file releases in the `dist/` folder
- Embeds dot-sourced scripts and file resources (e.g., `.txt`, `.html`) directly into the output
- Generates version numbers based on the most recent commit date for each script

### Code Formatting and Linting


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [microsoft/CSS-Exchange](https://github.com/microsoft/CSS-Exchange) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
