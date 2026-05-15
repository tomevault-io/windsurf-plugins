---
trigger: always_on
description: This is a **meta-repository** for Claude Code. It serves as a unified workspace with shared skills, commands, and an automated setup evaluator — so every team member gets consistent, high-quality AI assistance from day one.
---

## Overview

This is a **meta-repository** for Claude Code. It serves as a unified workspace with shared skills, commands, and an automated setup evaluator — so every team member gets consistent, high-quality AI assistance from day one.

**Note:** The "Agent Docs" listing below is auto-generated from frontmatter in markdown files in [`agent-docs/`](agent-docs/).

## **Critical Requirements**

<critical-requirements>
These requirements ALWAYS apply. Follow them throughout every task - some apply at task start, others when specific situations arise:

1. **Read relevant documentation first** - Before taking action on any task:
   - Review the [Agent Docs](#agent-docs) list below
   - Read ALL documents relevant to your task before proceeding
   - Re-evaluate after encountering failures or gaining new insights—additional documents may become relevant

2. **Use `uv` for Python execution** - Execute all scripts and tools with `uv run <script/tool>`. This workspace exclusively uses `uv` for package and environment management. For scripts with inline dependencies (PEP 723), run them directly (`uv run script.py`) to ensure dependencies resolve correctly.

3. **Use available skills** - Check if relevant skills are available in your environment for your tasks, and if there are - utilize them.

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

For more information, see [`agent-docs/README.md`](agent-docs/README.md).

### Agent Docs

<agent-docs-items>
<doc path="agent-docs/codebase-onboarding.md">
<name>Codebase Onboarding</name>
<description>Systematic onboarding to unfamiliar codebases. Analyzes project structure, tech stack, conventions, entry points, and data flow to produce an onboarding guide. Use when joining a new project or helping a team member get started.</description>
<when-to-read>When exploring an unfamiliar repository for the first time. When generating onboarding docs for a new team member. When analyzing project structure, conventions, dependencies, or entry points of a codebase you haven't worked in before.</when-to-read>
</doc>
<doc path="agent-docs/deep-research.md">
<name>Deep Research</name>
<description>Multi-source deep research with citations. Use when the user wants thorough research on any topic — technology evaluation, library comparison, algorithm analysis, literature review, or competitive analysis. Searches the web, synthesizes findings, and delivers cited reports.</description>
<when-to-read>When the user asks for research, comparison, or evaluation of technologies, libraries, algorithms, or approaches. When producing a literature review, competitive analysis, or trade-off assessment that requires multiple web sources.</when-to-read>
</doc>
<doc path="agent-docs/mcp-patterns.md">
<name>Mcp Patterns</name>
<description>Patterns for building, securing, and consuming MCP (Model Context Protocol) servers. Covers schema-first design, authentication, input validation, audit logging, and security best practices.</description>
<when-to-read>When building, modifying, or debugging an MCP server. When defining MCP tool schemas, configuring MCP authentication, or reviewing MCP security. When a .mcp.json file is present and the user is working on MCP integration.</when-to-read>
</doc>
<doc path="agent-docs/subagent-driven-development.md">
<name>Subagent-Driven Development Guide</name>
<description>Execute implementation plans by dispatching a fresh subagent per task with two-stage review (spec compliance, then code quality). Power-user workflow for complex multi-task implementations.</description>
<when-to-read>When executing implementation plans with many independent tasks in the current session. When you need isolated context per task to avoid pollution.</when-to-read>
</doc>
<doc path="agent-docs/workspace-development.md">
<name>Workspace Development Guide</name>
<description>How to modify, configure, and extend this workspace. Covers configuration, package management, creating agent-docs/skills/commands, tool discovery, and session hooks.</description>

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [redhat-community-ai-tools/claude-code-setup-evaluator](https://github.com/redhat-community-ai-tools/claude-code-setup-evaluator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
