---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

mach10 defines a structured development methodology for agentic coding and provides a Claude Code plugin that implements it. The methodology centers on fresh sessions for each step, GitHub as the inter-session persistence layer, staged implementation, and iterative review-fix cycles. The plugin codifies this into 14 slash commands covering the complete issue-to-merge lifecycle.

## Architecture

This is a pure plugin definition -- no build system, no runtime, no tests. Claude Code reads the command files directly.

```
.claude-plugin/plugin.json   # Plugin manifest (name, version, author)
agents/                       # Specialized agent definitions (markdown with YAML frontmatter)
commands/                     # Slash command definitions (markdown with YAML frontmatter)
```

Each command file is a self-contained workflow specification with:
- **YAML frontmatter**: `description`, `argument-hint`, `allowed-tools`, `model`
- **Markdown body**: Step-by-step instructions Claude Code follows when the command is invoked

## Command Structure

**Issue workflow:** `issue-assessment`, `issue-plan`, `issue-plan-review`, `issue-implement`, `issue-create`
**PR workflow:** `pr-create`, `pr-review`, `pr-review-fix`, `pr-ci-fix`, `doc-review`, `pr-pre-merge`, `pr-merge`
**Utilities:** `push`, `test-audit`

## Key Design Decisions

- **Thin orchestration**: Commands gather context and delegate to `feature-dev` and `pr-review-toolkit` plugins via the Skill tool rather than reimplementing workflows.
- **Single-concern sessions**: Review and fix are separate commands to preserve context budget for implementation. Each command is designed to run in a fresh CLI session.
- **GitHub as backbone**: Plans, reviews, and progress are posted as issue/PR comments so context persists across sessions.
- **Safe defaults**: No `git add -A`, no staging of secrets, no force-pushes, branch deletion uses `-d` flag.

## GitHub Comment Formatting

When composing content for GitHub comments or issue bodies:
- Do not use bare `#<number>` notation for numbered items (findings, suggestions, stages) -- GitHub auto-links `#<number>` to issues and PRs.
- Use plain words instead: "finding 3", "suggestion 3", "stage 2".
- Reserve `#<number>` exclusively for intentional GitHub issue/PR references (e.g., `Fixes #55`).

## Plugin Dependencies

- `gh` CLI (GitHub authentication required)
- `feature-dev` plugin (used by `issue-plan`, `issue-implement`, `pr-review-fix`, `pr-ci-fix`)
- `pr-review-toolkit` plugin (used by `pr-review`)

## Writing Commands

When adding or modifying commands, follow the existing pattern:
- YAML frontmatter must include `description` and `argument-hint`
- Set `allowed-tools` to the minimum set needed (principle of least privilege)
- Set `model: opus` for commands requiring deep reasoning (reviews, planning, implementation)
- End commands with a "next step" suggestion that includes `/clear` guidance for session boundaries (omit `/clear` when the next command needs session context, e.g., `/mach10:push` after implementation)
- Use `gh` CLI for all GitHub operations (issues, PRs, comments, CI logs)
- Commands that modify code delegate to `/feature-dev:feature-dev` via the Skill tool
- Commands that need codebase exploration or architecture analysis can use the Task tool with `subagent_type` to invoke agents from dependent plugins (e.g., `feature-dev:code-explorer`, `feature-dev:code-architect`)
- Commands that review code delegate to `/pr-review-toolkit:review-pr` via the Skill tool
- **Repo structure sync**: The repository structure diagram is duplicated between `CLAUDE.md` (Architecture section) and `README.md` (Repository structure section). Any change to the structure block must be applied to both locations.
- **Contributing-guide lookup sync**: The lookup pattern (`CONTRIBUTING.md` → `DEVELOPMENT.md` → `.github/CONTRIBUTING.md`, first-match-stop) is duplicated in three command files: `commands/issue-plan.md` (Step 2a), `commands/issue-plan-review.md` (Step 3a), and `commands/pr-pre-merge.md` (contributing guide lookup). `README.md` (the "Customizing with CONTRIBUTING.md" section) also documents this feature. Any change to the lookup logic must be applied to all four locations.
- **Plan marker sync**: The `<!-- mach10-plan -->` HTML marker is used to reliably locate implementation plan comments. The marker is emitted by `commands/issue-plan.md` (Step 6) and `commands/issue-plan-review.md` (revised plan posting). It is consumed by `commands/issue-implement.md` (Step 3), `commands/issue-plan-review.md` (Step 2), and `agents/feature-completeness-checker.md` (Step 1). Any change to the marker convention must be applied to all locations.
- **Issue-reference pattern sync**: The list of issue reference patterns (`Fixes #N`, `Closes #N`, `Resolves #N`, `Part of #N`, `Issue #N`, bare `#N`) is duplicated between `commands/pr-review.md` (Step 2, Skill invocation instruction) and `agents/feature-completeness-checker.md` (Step 1, "Detect the linked issue"). Any change to the pattern list must be applied to both locations.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LeanAndMean/mach10](https://github.com/LeanAndMean/mach10) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
