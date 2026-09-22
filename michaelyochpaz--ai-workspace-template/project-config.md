---
trigger: always_on
description: This is a **meta-repository** that aggregates related repositories using git submodules. It serves as a unified workspace for AI agents (LLMs with specific roles and tools, designed to address specific task types) to operate across the project ecosystem.
---

## Overview

This is a **meta-repository** that aggregates related repositories using git submodules. It serves as a unified workspace for AI agents (LLMs with specific roles and tools, designed to address specific task types) to operate across the project ecosystem.

**Note:** The "Agent Docs" listing below is auto-generated from frontmatter in markdown files in [`agent-docs/`](agent-docs/).

## **Critical Requirements**

<critical-requirements>
These requirements ALWAYS apply. Follow them throughout every task - some apply at task start, others when specific situations arise:

1. **Read relevant documentation first** - Before taking action on any task:
   - Review the [Agent Docs](#agent-docs) list below and read all documents relevant to your task before proceeding
   - When the task scope depends on an external resource (Jira ticket, PR, error log), fetch that resource first to understand the task, then select and read relevant documents based on what you learned
   - Re-evaluate relevance after encountering failures or gaining new insights — additional documents may become relevant

2. **Use `uv` for Python execution** - Execute all scripts and tools with `uv run <script/tool>`. This workspace exclusively uses `uv` for package and environment management. For scripts with inline dependencies (PEP 723), run them directly (`uv run script.py`) to ensure dependencies resolve correctly.

3. **Use available skills** - Review all available skill descriptions and match them to the task's underlying objective, including when decomposing into subtasks. Reference matching skills as available resources. Use matching skills instead of manual approaches.

4. **Verify before diagnosing** - When analyzing failures or investigating issues, use documentation and available tools to verify facts. Provide diagnoses **ONLY after verification**.

Violating these requirements results in incomplete solutions, wasted effort, and task failures.
</critical-requirements>


## Modular Documentation System

The [`agent-docs/`](agent-docs/) directory contains modular, task-focused documentation that provides crucial information, context, and instructions for various tasks relevant to this workspace.

**Documentation review workflow:**
1. **ALWAYS** review the "Agent Docs" list below when starting any task
2. Identify documents relevant to your task based on their descriptions and "When to read" sections
3. **Read ALL identified relevant documents before proceeding**
4. **Re-evaluate relevance whenever you gain new insights** during the task - when discoveries change your understanding, immediately check if additional documents are now relevant

When creating or modifying docs, read [`agent-docs/README.md`](agent-docs/README.md) first.

### Agent Docs

<agent-docs-items>
<doc path="agent-docs/workspace-development.md">
<name>Workspace Development Guide</name>
<description>How to modify, configure, and extend this workspace. Covers configuration, AGENTS.md generation, package management, creating agent-docs/skills/commands, tool discovery, and session hooks.</description>
<when-to-read>When modifying workspace infrastructure or configuration. When creating or editing agent-docs, skills, or commands. When updating AGENTS.md, README.md, or AGENTS.project.md. When adding dependencies or changing ai-workspace.toml. When adding session hook support for a new AI tool.</when-to-read>
</doc>
</agent-docs-items>

## Agent Resources Overview

In addition to environment-provided capabilities (e.g., tools, MCPs, Skills), this workspace defines project-specific resources:

- **Documentation** ([`agent-docs/`](agent-docs/)) — AI reads when relevant to tasks
- **Skills** ([`skills/`](skills/)) — Reusable agent capabilities with scripts and instructions
- **Commands** ([`commands/`](commands/)) — Human invokes via `/command`; AI receives prompt

## Configuration

This workspace is configured via `ai-workspace.toml` at the repository root. After changing configuration, agent docs, or `AGENTS.project.md`, regenerate workspace files:
```bash
uv run .ai-workspace/scripts/align-workspace.py
```

For configuration reference, see [`.ai-workspace/README.md`](.ai-workspace/README.md).

## Pre-commit Hooks

Pre-commit hooks are configured to maintain code quality and documentation consistency.

**Setup (if not already installed):**
```bash
uv run pre-commit install
```

**Recommended workflow:** After completing tasks, run pre-commit to validate the workspace:
```bash
uv run pre-commit run --all-files
```

Pre-commit verifies that `AGENTS.md` and feature directories are in sync with configuration. If verification fails, run the alignment script above and stage the updated files.

## Temporary Files

The `.tmp/` directory is git-ignored and available for transient artifacts, design documents, logs, or intermediate files.

**When writing files to `.tmp/`:** First create a task-specific subdirectory:

```bash
uv run .ai-workspace/scripts/mktmpdir.py [name]
```

The script outputs the created path. If the directory exists, returns the existing path.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MichaelYochpaz/ai-workspace-template](https://github.com/MichaelYochpaz/ai-workspace-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-22 -->
