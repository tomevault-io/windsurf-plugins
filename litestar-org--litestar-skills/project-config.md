---
trigger: always_on
description: This file is loaded by every AI agent CLI that consumes this repo: Claude Code, Antigravity CLI, Codex CLI, Cursor, OpenCode, VS Code/Copilot, and others supporting the [agentskills.io](https://agentskills.io) standard.
---

# `litestar-skills` Agent Context

This file is loaded by every AI agent CLI that consumes this repo: Claude Code, Antigravity CLI, Codex CLI, Cursor, OpenCode, VS Code/Copilot, and others supporting the [agentskills.io](https://agentskills.io) standard.

## Mission

`litestar-skills` is a curated, opinionated collection of agent skills, plugins, subagents, slash commands, and MCP servers for the **Litestar** framework and its first-party ecosystem. Agents working in this repo are authoring or maintaining content that will be consumed by other agents to guide Litestar application development.

Style baseline for every skill here: [`skills/litestar-styleguide/`](skills/litestar-styleguide/SKILL.md).

## Agent Conduct

- **Terse, imperative, authoritative tone.** No hedging. State the preferred choice.
- **Litestar-first-party bias.** Prefer `litestar-granian`, `litestar-saq`, `sqlspec`, `advanced-alchemy`, `msgspec` over generic alternatives in examples and recommendations.
- **Minimal targeted changes.** Do not make opportunistic cleanup edits without approval.
- **Canonical commands via Make.** Always prefer `make <target>` over raw tool invocations. See [Development Commands](#development-commands).
- **Never silently descope.** If a task is larger than expected, refine the plan or ask how to prioritize.
- **No blame language.** Describe problems factually; offer the smallest useful next step.

## Skill Authoring Rules

Every `SKILL.md` MUST follow these conventions:

1. **YAML frontmatter** with `name` (kebab-case, matches directory) and trigger-only `description` (starts with `Auto-activate for` or `Use when`, includes concrete file/import/API signals, ends with `Not for X — why`, and contains no process summary such as "Produces ...").
2. **XML-tagged sections** in this order: Code Style Rules → Quick Reference → `<workflow>` → `<guardrails>` → `<validation>` → `<example>` → References Index → Official References → Shared Styleguide Baseline.
3. **Match-your-stack**: when multiple valid libraries / backends / patterns exist for a concern (data access, DI, background tasks, Channels backend, settings, serialization, deployment target), present all options and help the user pick based on what's already in their project. Never force one path.
4. **Litestar code-sample conventions** (full detail in [`skills/litestar-styleguide/`](skills/litestar-styleguide/SKILL.md)):
   - PEP 604 unions (`T | None`), never `Optional[T]`
   - `from __future__ import annotations` is a library-author guardrail, not a consumer rule — application code MAY use it; library code that defines runtime-introspected types avoids it
   - Google-style docstrings, async all I/O
   - `msgspec` over Pydantic in Litestar contexts (unless the project already uses Pydantic)
   - `advanced-alchemy` / `sqlspec` over raw SQLAlchemy (match the project's chosen stack)

## Supported Hosts

Document hosts by the artifacts this repo ships. Do not describe hosts as compatibility tiers.

| Host | Entry Point | Notes |
| --- | --- | --- |
| **Claude Code** | `.claude-plugin/plugin.json` + `.claude-plugin/marketplace.json` + `.claude-plugin/agents/*.md` | Full plugin with skills, commands, agents, hooks. |
| **Antigravity CLI** | `plugin.json` + `hooks.json` + `agents/*.md` + `skills/` + `hooks/` | Google CLI plugin with Markdown subagent templates and SessionStart hooks. |
| **Codex CLI** | `.codex-plugin/plugin.json` + `.codex/agents/*.toml` + `.codex/config.toml` | Custom agents ship as pure TOML (tools inherited from session). |
| **OpenCode** | `.opencode/plugins/litestar.js` + `.opencode/agents/*.md` + native `.claude/skills/` / `.agents/skills/` reads | JS plugin wrapper + dict-schema agents. |
| **Cursor** | `.cursor-plugin/plugin.json` | Hooks via `hooks/hooks-cursor.json`. |
| **VS Code / Copilot** | User adds path to `chat.skillsLocations` | Raw SKILL.md tree; no wrapper extension in v0.1. |
| **OpenClaw** | `.agents/skills/` + `AGENTS.md` | Consumes generic Agent Skills tree without extra config. |

## File Resolution

| Resource | Location |
| --- | --- |
| Skills | `skills/<skill-name>/SKILL.md` |
| Slash commands | `commands/<prefix>/<command>.toml` |
| Subagents (Antigravity CLI) | `agents/<agent-name>.md` (`tools` as YAML list of Antigravity tool names) |
| Subagents (Claude Code) | `.claude-plugin/agents/<agent-name>.md` (`tools` as comma-separated string of Claude tool names) |
| Subagents (Codex CLI) | `.codex/agents/<agent-name>.toml` (pure TOML; `developer_instructions` holds the prompt; no top-level `tools` — inherited from session `config.toml`) |
| Subagents (OpenCode) | `.opencode/agents/<agent-name>.md` (`tools` as dict mapping + `mode: subagent`) |
| MCP servers | `mcp-servers/<server-name>/` |
| Hooks | Root `hooks.json` for Antigravity CLI; `hooks/hooks-<host>.json` for Claude/Cursor/Codex; shared runtime in `hooks/session-start.{sh,ps1,js}` + `hooks/lib/{detect-env.{sh,ps1,js},_detector.py,skill-map.json}` |
| Templates | `templates/skill-template/` |

## Harness Names

Keep package identity, skill namespace, command syntax, and agent syntax separate:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [litestar-org/litestar-skills](https://github.com/litestar-org/litestar-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-25 -->
