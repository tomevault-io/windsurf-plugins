---
trigger: always_on
description: **Closing the intent-to-code chasm**
---

# Intent Integrity Kit

**Closing the intent-to-code chasm**

## Overview

Intent Integrity Kit (IIKit) preserves your intent from idea to implementation through specification-driven development with cryptographic verification. Compatible with Claude Code, Codex, Gemini, OpenCode, and GitHub Copilot.

## Intent Integrity Kit Workflow

This project uses specification-driven development. The phases are:

**Utility:** `/iikit-core` - Initialize project (git/GitHub setup, PRD seeding), check status, select active feature, show help (run `init` before starting)
**Utility:** `/iikit-clarify` - Resolve ambiguities in any artifact (spec, plan, checklist, testify, tasks, constitution) — runnable after any phase
**Utility:** `/iikit-bugfix` - Report and fix bugs without full specification workflow

0. `/iikit-00-constitution` - Define project governance principles
1. `/iikit-01-specify` - Create feature specification from natural language
2. `/iikit-02-plan` - Create technical implementation plan
3. `/iikit-03-checklist` - Generate domain-specific quality checklists
4. `/iikit-04-testify` - Generate test specifications (TDD support, optional unless constitutionally required)
5. `/iikit-05-tasks` - Generate task breakdown
6. `/iikit-06-analyze` - Validate cross-artifact consistency
7. `/iikit-07-implement` - Execute implementation
8. `/iikit-08-taskstoissues` - Export tasks to GitHub Issues

**Never skip phases.** Each `/iikit-*` command validates its prerequisites.

Read `FRAMEWORK-PRINCIPLES.md` for this framework's development principles.

## Project Structure

```text
.claude/skills/              # Development layout (source of truth)
  iikit-core/              # Core skill with scripts and templates
    scripts/bash/            # Bash scripts for all skills
    scripts/powershell/      # PowerShell scripts for Windows
    templates/               # Framework templates (do not edit)
    references/              # Shared reference docs (do not edit)
  iikit-00-constitution/   # Constitution skill
  iikit-01-specify/        # Specification skill
  ...                        # Other iikit-XX-* skills
.codex/skills/               # Symlink -> .claude/skills
.gemini/skills/              # Symlink -> .claude/skills
.opencode/skills/            # Symlink -> .claude/skills
.github/copilot-instructions.md  # Symlink -> AGENTS.md (GitHub Copilot)

CONSTITUTION.md              # Project governance (content-agnostic, lives at root)
PREMISE.md                   # App-wide context: what, who, why, domain, scope

.specify/
  context.json               # Feature state between skill invocations
  active-feature             # Sticky active feature selection (survives restarts)
  evals/                     # Tessl eval results (fetched, not committed)

specs/                       # Feature specifications (created per feature)
  NNN-feature-name/
    spec.md                  # Feature specification
    plan.md                  # Implementation plan
    tasks.md                 # Task breakdown
    research.md              # Research findings
    data-model.md            # Data model
    quickstart.md            # Quick start guide
    contracts/               # API contracts
    checklists/              # Quality checklists
    tests/                   # Test specifications (created by /iikit-04-testify)
      features/              # Gherkin .feature files (locked by assertion hash)
```

> **Distribution note**: The `.claude/skills/` layout above is the development structure. In development, skills reference shared files in `iikit-core/` via `../iikit-core/` paths. During CI publish, these are copied into each skill and links are rewritten for self-containment. Use `tessl install tessl-labs/intent-integrity-kit` for correct setup — cloning the repo directly does not produce self-contained skills.

## Commands

```bash
# Make scripts executable (if needed)
chmod +x .claude/skills/iikit-core/scripts/bash/*.sh

# Detect git/GitHub environment
.claude/skills/iikit-core/scripts/bash/git-setup.sh --json

# Check prerequisites for a feature
.claude/skills/iikit-core/scripts/bash/check-prerequisites.sh --json

# Create a new feature
.claude/skills/iikit-core/scripts/bash/create-new-feature.sh --json "Feature description"

# Select active feature (multi-feature projects)
.claude/skills/iikit-core/scripts/bash/set-active-feature.sh --json <selector>
```

## Skills Available

| Skill | Command | Description |
|-------|---------|-------------|
| Core | `/iikit-core` | Initialize project (git/GitHub setup, PRD seeding), check status, select feature, show help |
| Clarify | `/iikit-clarify` | Resolve ambiguities in any artifact (utility, runnable after any phase) |
| Bugfix | `/iikit-bugfix` | Report and fix bugs without full specification workflow |
| Constitution | `/iikit-00-constitution` | Create project governance principles |
| Specify | `/iikit-01-specify` | Create feature spec from description |
| Plan | `/iikit-02-plan` | Create technical implementation plan |
| Checklist | `/iikit-03-checklist` | Generate quality checklists |
| Testify | `/iikit-04-testify` | Generate test specs (TDD support) |
| Tasks | `/iikit-05-tasks` | Generate task breakdown |
| Analyze | `/iikit-06-analyze` | Validate cross-artifact consistency |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [intent-integrity-chain/kit](https://github.com/intent-integrity-chain/kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
