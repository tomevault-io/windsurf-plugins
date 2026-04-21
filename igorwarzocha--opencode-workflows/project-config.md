---
trigger: always_on
description: This is the **Opencode Workflows** repository - a collection of Opencode-based command templates and workflow patterns for building sophisticated command-based projects. The repository contains multiple workflow examples and templates that demonstrate different approaches to command architecture and tool integration.
---

## Repository Overview

This is the **Opencode Workflows** repository - a collection of Opencode-based command templates and workflow patterns for building sophisticated command-based projects. The repository contains multiple workflow examples and templates that demonstrate different approaches to command architecture and tool integration.

<instructions>
## Verification Commands
This is a **workflow repository**, not a traditional application. Verification focuses on repository integrity and structural validity.

- **Audit Repository**: `/audit-repo` (Validates structure and configs via `audit_repo.py`)
- **Sync Documentation**: `/sync-docs` (Reports inventory and suggests doc updates via `sync_docs.py`)
- **Full Maintenance**: `/maintain-repo` (Runs full audit and sync cycle)

**Note**: There are no traditional `npm test`, `cargo build`, or `tsc` commands at the root level.
</instructions>

<rules>
## Process Constraints
- MUST NOT run long-running/blocking processes (dev servers, watch modes)
- Dev servers/background processes are USER's responsibility
- MUST use one-shot commands for verification (audit, sync, scripts)

## Coding Conventions
- **RFC 2119**: MUST use uppercase keywords (MUST, SHOULD, MAY) for requirements in agents and commands.
- **XML Structure**: MUST use XML tags (`<instructions>`, `<rules>`, etc.) to wrap logic blocks.
- **Modularity**: Files SHOULD NOT exceed 200 lines; functions SHOULD NOT exceed 40 lines.
- **Barrel Files**: Every module directory MUST have an `index.ts` (per `@coding-ts` guidelines).
</rules>

## Current Workflows

### Agent Templates Catalog
A focused collection of reusable agent prompts and orchestration patterns:
- **repo-maintainer**: Repository health custodian (audits, doc sync). **NOTE: This root agent is specific to the Opencode-Workflows repo.**
- **fast**: High-speed workhorse for trivial edits, running known commands, and simple file lookups.
- **smart**: Senior developer and architect for complex bug hunting, codebase refactoring, and verified implementation.
- **repo-navigator-creator**: Produces lean AGENTS.md navigation guides
- **subagent-orchestrator**: Dispatches specialized agents and manages execution plans
- **openspec-orchestrator**: Enforces strict OpenSpec formatting/validation and orchestrates subagents

Agents are designed for global installation in `~/.config/opencode/agent/` for reuse across projects.

### AI Research Tools
- AI search integration tools and patterns are demonstrated within the specialized agent packs (see `agents/opencode-configurator/`).
- Integrates with external AI services (e.g., Perplexica, OpenAI) via specialized skills.

## Architecture Patterns

### Command Structure
Commands follow Opencode's built-in `/commands` patterns (see opencode.ai/docs/commands):
- YAML frontmatter with descriptions
- Arguments section explaining $ARGUMENTS handling
- Step-by-step execution instructions
- Integration with external utilities

### Tool Integration
Commands can integrate with various external tools:
- **Script files**: JavaScript, Python, Bash executables
- **Bash tools**: System utilities, package managers, development tools
- **API endpoints**: Any curlable REST APIs or webhooks
- **Natural language workflows**: Pure LLM-driven processes

### Agent Structure
Agents follow Opencode's agent patterns with YAML frontmatter:
- **Description**: Clear guidance on when to use each agent
- **Mode**: Operation mode and tool constraints (read-only vs write)
- **Instruction Blocks**: LLM-optimized checklists and workflows
- **Global Installation**: Designed for reuse across projects via `~/.config/opencode/agent/`

<routing>
## Task Navigation
| Task | Entry Point | Key Files |
|------|-------------|-----------|
| Create AGENTS.md | /init | `agents/repo-navigator/` |
| Security Review | /security-review | `agents/security-reviewer/` |
| PRD Planning | /prd | `agents/parallel-PRD/` |
| Maintain Repo | /maintain-repo | `.opencode/command/` |
| Create Plugin | /create-plugin | `agents/create-opencode-plugin/` |
</routing>

### Configuration System
- `example-opencode.json` templates for Opencode configuration. Demonstrates disabling the legacy `general` subagent in favor of `fast`/`smart` splitting to optimize model usage:
  ```json
  "subagents": {
    "general": {
      "disable": true
    }
  }
  ```
- References to `AGENTS.md` in instructions arrays
- `package.json` in `.opencode/` for Opencode plugin dependencies
- Follows Opencode schema standards

## Model Requirements

Repository intelligence requires models with strong:
- **Context Management**: Maintaining project-level instructions throughout session
- **Instruction Following**: Precise execution of multi-step workflows
- **Agentic Capabilities**: Understanding when/how to use available commands and agents

**Recommended Models**: GPT-5.2, Claude 4.5, Gemini 3.

## Repository Structure

```
at/                          # Universal engineering guidelines (@coding-ts)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [IgorWarzocha/Opencode-Workflows](https://github.com/IgorWarzocha/Opencode-Workflows) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
