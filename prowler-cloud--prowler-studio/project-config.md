---
trigger: always_on
description: This document covers the architecture of Prowler Studio and the coding standards for developing its agents. Read [System Architecture](#system-architecture) to understand how the pieces fit together; read the development best practices that follow before adding or modifying an agent.
---

# Agent Development Guide

This document covers the architecture of Prowler Studio and the coding standards for developing its agents. Read [System Architecture](#system-architecture) to understand how the pieces fit together; read the development best practices that follow before adding or modifying an agent.

## Table of Contents

**System Architecture**

- [Overview](#overview)
- [Project Structure](#project-structure)
- [Agents](#agents)
- [ChecKreatorAgent Flow](#checkreatoragent-flow)
- [Shared Tools](#shared-tools)
- [Logging & Observability](#logging--observability)
- [Main CLI Orchestration](#main-cli-orchestration)
- [Adding a New Agent](#adding-a-new-agent)

**Development Best Practices**

- [Architecture Principles](#architecture-principles)
- [Code Organization](#code-organization)
- [Method Decomposition](#method-decomposition)
- [Type Safety](#type-safety)
- [Named Parameters](#named-parameters)
- [Return Types with Pydantic](#return-types-with-pydantic)
- [Constants Management](#constants-management)
- [File Structure](#file-structure)
- [Examples](#examples)
- [Checklist for New Agents](#checklist-for-new-agents)

---

## System Architecture

### Overview

Prowler Studio uses the Claude Agent SDK to automate the creation of security checks for Prowler. The architecture separates each task (implementation, testing, compliance mapping, review, PR creation) into an independent agent. Agents run sequentially, with built-in verification and error-correction loops between them.

### Project Structure

```
prowler_studio/
├── src/
│   ├── core/
│   │   ├── main.py              # CLI entry point
│   │   └── exceptions.py        # Custom exceptions
│   ├── agents/
│   │   ├── base.py              # Agent base class
│   │   ├── implementation/      # ChecKreatorAgent for check creation
│   │   │   ├── agent.py         # Main agent implementation
│   │   │   ├── models.py        # Data models
│   │   │   └── prompts/         # Jinja2 prompt templates
│   │   ├── testing/             # TestingAgent for test generation
│   │   ├── review/              # ReviewAgent for code review
│   │   ├── compliance_mapping/  # ComplianceMappingAgent
│   │   └── pr_creation/         # PRCreationAgent for PR workflow
│   ├── tools/                   # Shared tools
│   │   ├── git.py               # Git operations and worktree management
│   │   ├── prowler.py           # Prowler-specific tools
│   │   ├── skills.py            # AI skills setup
│   │   ├── jira.py              # Jira URL parsing
│   │   ├── jira_client.py       # Jira API client
│   │   ├── github_issue.py      # GitHub issue URL parsing
│   │   ├── github_client.py     # GitHub REST API client
│   │   └── models.py            # Tool data models
│   └── utils/                   # Utilities
│       ├── prompts.py           # Prompt loading utilities
│       └── logging.py           # Logging utilities (agent output + tool calls)
└── pyproject.toml               # Project configuration
```

### Agents

Each agent is a self-contained unit that performs a specific task. Every agent inherits from the `Agent` base class in [src/agents/base.py](src/agents/base.py), implements `async run()`, and returns a typed result object.

**Current Agents:**

- **ChecKreatorAgent** ([src/agents/implementation/agent.py](src/agents/implementation/agent.py)) — Creates Prowler checks from tickets with automated verification.
- **TestingAgent** ([src/agents/testing/agent.py](src/agents/testing/agent.py)) — Generates and runs tests for checks with an auto-fix loop.
- **ReviewAgent** ([src/agents/review/agent.py](src/agents/review/agent.py)) — Reviews check implementations for quality and best practices.
- **ComplianceMappingAgent** ([src/agents/compliance_mapping/agent.py](src/agents/compliance_mapping/agent.py)) — Analyzes checks and adds compliance framework mappings.
- **PRCreationAgent** ([src/agents/pr_creation/agent.py](src/agents/pr_creation/agent.py)) — Commits changes and creates pull requests.

### ChecKreatorAgent Flow

The implementation agent follows this workflow:

1. **Setup** — Load prompts and configure the Claude Agent SDK with MCP tools.
2. **Implementation** — Claude agent creates the check based on ticket requirements.
3. **Discovery** — Automatically detect the created check from git changes.
4. **Verification Loop** (up to 5 attempts):
   - Run `prowler <provider> --list-checks` to verify the check loads.
   - If verification fails, provide error feedback to Claude.
   - Claude fixes the issues and verification runs again.
5. **Result** — Return success/failure with check details.

Key features:

- Uses Claude Agent SDK with a custom MCP server that exposes the `mkcheck` tool.
- Jinja2 templates for prompts in [src/agents/implementation/prompts/](src/agents/implementation/prompts/).
- Typed result models: `CheckImplementationResult`, `CheckDiscoveryResult`, `CheckVerificationResult`.

### Shared Tools

#### Git Tools ([src/tools/git.py](src/tools/git.py))


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [prowler-cloud/prowler-studio](https://github.com/prowler-cloud/prowler-studio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-02 -->
