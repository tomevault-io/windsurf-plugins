---
trigger: always_on
description: **Atomic Spec** is a governance framework for AI-driven development — a customized fork of [GitHub Spec Kit](https://github.com/github/spec-kit) that enforces the Atomic Traceability Model (gated, atomic, context-pinned phases). See [`atomic-traceability-model.md`](./atomic-traceability-model.md) for the governance model.
---

# AGENTS.md — Adding a New AI Agent to Atomic Spec

## About Atomic Spec and the `atomicspec` CLI

**Atomic Spec** is a governance framework for AI-driven development — a customized fork of [GitHub Spec Kit](https://github.com/github/spec-kit) that enforces the Atomic Traceability Model (gated, atomic, context-pinned phases). See [`atomic-traceability-model.md`](./atomic-traceability-model.md) for the governance model.

The **`atomicspec` CLI** (PyPI package: `atomic-spec`) bootstraps projects with the framework. It sets up the necessary directory structures, templates, and AI-agent integrations to support the four-phase Specify → Plan → Tasks → Implement workflow.

The framework supports 17+ AI coding assistants across two tiers:

- **Supported tier** (wired end-to-end, exercised on every release): `claude`, `gemini`, `copilot`, `cursor-agent`, `windsurf`
- **Experimental tier** (template-enforced governance, best-effort triage): `qwen`, `opencode`, `codex`, `kilocode`, `auggie`, `codebuddy`, `qoder`, `roo`, `q`, `amp`, `shai`, `bob`

Agent matching for subagents is **dynamic**: keyword overlap between feature descriptions and YAML frontmatter `description` fields — no hard-coded agent lists in command templates.

This guide explains how to add a new agent to the supported or experimental tier.

---

## General practices

- Any changes to `src/specify_cli/__init__.py` require a version rev in `pyproject.toml` and an entry in `CHANGELOG.md`.
- Agent metadata (`AGENT_CONFIG` and `ATOMIC_SPEC_COMMANDS`) lives in `src/specify_cli/_config.py` — a stdlib-only module that the release workflow imports via `importlib.util` without installing CLI dependencies.

## Adding New Agent Support

This section explains how to add support for new AI agents/assistants to the Specify CLI. Use this guide as a reference when integrating new AI tools into the Spec-Driven Development workflow.

### Overview

Specify supports multiple AI agents by generating agent-specific command files and directory structures when initializing projects. Each agent has its own conventions for:

- **Command file formats** (Markdown, TOML, etc.)
- **Directory structures** (`.claude/commands/`, `.windsurf/workflows/`, etc.)
- **Command invocation patterns** (slash commands, CLI tools, etc.)
- **Argument passing conventions** (`$ARGUMENTS`, `{{args}}`, etc.)

### Current Supported Agents

The CLI accepts **17 agent keys**, but they are **NOT all equally validated**. Per README "AI coding agents supported," `Supported` = wired end-to-end and exercised on every release; `Experimental` = templates install correctly and the agent key is accepted, but agent-specific wiring is not exhaustively tested.

| Agent                      | Tier             | Directory              | Format   | CLI Tool        | Description                 |
| -------------------------- | ---------------- | ---------------------- | -------- | --------------- | --------------------------- |
| **Claude Code**            | **Supported**    | `.claude/commands/`    | Markdown | `claude`        | Anthropic's Claude Code CLI |
| **Gemini CLI**             | **Supported**    | `.gemini/commands/`    | TOML     | `gemini`        | Google's Gemini CLI         |
| **GitHub Copilot**         | **Supported**    | `.github/agents/`      | Markdown | N/A (IDE-based) | GitHub Copilot in VS Code   |
| **Cursor**                 | **Supported**    | `.cursor/commands/`    | Markdown | `cursor-agent`  | Cursor CLI                  |
| **Windsurf**               | **Supported**    | `.windsurf/workflows/` | Markdown | N/A (IDE-based) | Windsurf IDE workflows      |
| **Qwen Code**              | *Experimental*   | `.qwen/commands/`      | TOML     | `qwen`          | Alibaba's Qwen Code CLI     |
| **opencode**               | *Experimental*   | `.opencode/command/`   | Markdown | `opencode`      | opencode CLI                |
| **Codex CLI**              | *Experimental*   | `.codex/commands/`     | Markdown | `codex`         | Codex CLI                   |
| **Kilo Code**              | *Experimental*   | `.kilocode/rules/`     | Markdown | N/A (IDE-based) | Kilo Code IDE               |
| **Auggie CLI**             | *Experimental*   | `.augment/rules/`      | Markdown | `auggie`        | Auggie CLI                  |
| **Roo Code**               | *Experimental*   | `.roo/rules/`          | Markdown | N/A (IDE-based) | Roo Code IDE                |
| **CodeBuddy CLI**          | *Experimental*   | `.codebuddy/commands/` | Markdown | `codebuddy`     | CodeBuddy CLI               |
| **Qoder CLI**              | *Experimental*   | `.qoder/commands/`     | Markdown | `qoder`         | Qoder CLI                   |
| **Amazon Q Developer CLI** | *Experimental*   | `.amazonq/prompts/`    | Markdown | `q`             | Amazon Q Developer CLI      |
| **Amp**                    | *Experimental*   | `.agents/commands/`    | Markdown | `amp`           | Amp CLI                     |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Chappygo-OS/Atomic-Spec](https://github.com/Chappygo-OS/Atomic-Spec) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-14 -->
