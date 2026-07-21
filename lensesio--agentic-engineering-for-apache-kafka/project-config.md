---
trigger: always_on
description: A drop-in collection of agent skills that turn AI agents and tools such as Claude Code and Cursor into Kafka-specialised engineering assistants. The repository **is** the plugin - the same `skills/` payload at the repo root is consumed by both the Claude Code marketplace (`.claude-plugin/`) and the Cursor marketplace (`.cursor-plugin/`).
---

# AGENTS.md

## Project Overview

A drop-in collection of agent skills that turn AI agents and tools such as Claude Code and Cursor into Kafka-specialised engineering assistants. The repository **is** the plugin - the same `skills/` payload at the repo root is consumed by both the Claude Code marketplace (`.claude-plugin/`) and the Cursor marketplace (`.cursor-plugin/`).

Maintained by [Lenses.io](https://lenses.io). Skills are MCP-agnostic by design but observed against the [Lenses MCP Server](https://github.com/lensesio/lenses-mcp); any Kafka MCP server that exposes an equivalent tool surface works.

This repo is a **Markdown skills payload**. It deliberately ships no source code, build system, tests, or runtime configuration - just `SKILL.md` files, their `references/`, plugin manifests, and documentation.

## Project Structure

```
.claude-plugin/
├── marketplace.json                  # Claude Code marketplace catalog (lists kafka-skills)
└── plugin.json                       # Claude Code plugin manifest
.cursor-plugin/
├── marketplace.json                  # Cursor marketplace catalog (lists kafka-skills)
└── plugin.json                       # Cursor plugin manifest (references assets/logo.svg)
assets/
└── logo.svg                          # Plugin logo (consumed by the Cursor manifest)
skills/                               # Shared SKILL.md payload (Claude Code + Cursor)
├── kafka-topic-audit/      (+ references/)
├── kafka-consumer-lag/     (+ references/)
├── kafka-perf-review/      (+ references/)
├── kafka-schema-review/    (+ references/)
├── kafka-security-audit/   (+ references/)
├── kafka-connector-review/ (+ references/)
├── kafka-dlq-review/       (+ references/)
└── kafka-python-client/    (+ references/)
AGENTS.md                             # Agent memory (this file)
README.md                             # Source of truth for end-user installation and usage
CONTRIBUTING.md                       # How to add a new skill, conventions, release process
TROUBLESHOOTING.md                    # Common issues (upload errors, triggering, MCP failures)
LICENSE                               # MIT
.gitignore                            # Includes .claude/ so Claude Code's per-user state stays out of git
```

For Claude Code, the eight skills ship as the `kafka-skills` plugin via the `lensesio` marketplace catalog. Install with `/plugin marketplace add lensesio/agentic-engineering-for-apache-kafka` then `/plugin install kafka-skills@lensesio`. Skills auto-trigger from their description; for explicit slash invocation use `/kafka-skills:<skill-name>`.

For Cursor, the same payload is exposed as a [Cursor plugin](https://cursor.com/docs/reference/plugins). Install through the [Cursor Marketplace](https://cursor.com/marketplace) or the `/add-plugin` command in the Cursor Agent. Cursor only requires `name` and `description` in skill frontmatter, so the same `SKILL.md` files serve both ecosystems without duplication. Claude-Code-only frontmatter fields (`allowed-tools`, `argument-hint`) are silently ignored by Cursor.

For the cross-tool [Skills CLI](https://github.com/vercel-labs/skills), the same payload is also installable via `npx skills add lensesio/agentic-engineering-for-apache-kafka`, which works with Cursor, Claude Code, Codex, OpenCode, Continue and 50+ other agents from one command. Discovery is driven by the `skills` array in [.claude-plugin/marketplace.json](.claude-plugin/marketplace.json) per the [plugin-manifest discovery format](https://github.com/vercel-labs/skills#plugin-manifest-discovery), so the same array gates both the Claude Code marketplace and `npx skills`. The repo is auto-indexed at [skills.sh/lensesio/agentic-engineering-for-apache-kafka](https://skills.sh/lensesio/agentic-engineering-for-apache-kafka); no manual submission is needed.

The published plugin payload deliberately stays narrow - just the eight Kafka skills and their `references/`. Hook and settings recipes (PostToolUse formatting, Stop-hook verification, pre-approved permissions, custom spinner verbs) belong in each consuming team's own `.claude/settings.json`, not in a portable skills plugin.

## Skill Structure Conventions

All skills follow the [Anthropic open standard](https://resources.anthropic.com/hubfs/The-Complete-Guide-to-Building-Skill-for-Claude.pdf) with progressive disclosure:

- **YAML frontmatter** includes `name`, `description` (with trigger phrases and negative triggers), `license`, `metadata` (author, version, mcp-server, category) and `compatibility` (for MCP skills)
- **SKILL.md body** contains workflow steps with expected output notes and validation gates, success criteria, examples and troubleshooting
- **`references/` directory** holds detailed lookup tables and domain rules loaded on demand (audit thresholds, config defaults, compatibility matrices)
- Each skill includes a **Success Criteria** section with quantitative and qualitative metrics

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lensesio/agentic-engineering-for-apache-kafka](https://github.com/lensesio/agentic-engineering-for-apache-kafka) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
