---
trigger: always_on
description: This file provides guidance to Claude Code when working with Founder OS.
---

# CLAUDE.md

This file provides guidance to Claude Code when working with Founder OS.

## Project Overview

Founder OS is a single-plugin AI automation ecosystem for SMB founders, built on Claude Code using the official Anthropic plugin format. 32 command namespaces cover email triage, meeting prep, report generation, CRM sync, and more — all invoked as `/founder-os:namespace:action`.

## Architecture

### Single Plugin Structure

Founder OS ships as one Anthropic-compatible plugin at the repository root:
```
founderOS/
├── .claude-plugin/
│   └── plugin.json           # Single plugin manifest
├── .mcp.json                 # MCP server connections
├── commands/
│   └── [namespace]/          # One directory per namespace
│       └── [action].md       # Slash command (e.g., commands/inbox/triage.md)
├── skills/
│   └── [namespace]/
│       └── [skill-name]/
│           └── SKILL.md      # Domain knowledge
├── agents/
│   └── [namespace]/          # Agent Teams (Priority 5 namespaces)
│       └── *.md
├── _infrastructure/          # Shared skills, templates, MCP configs
└── _templates/               # Scaffolding and examples
```

Commands are invoked as `/founder-os:namespace:action` (e.g., `/founder-os:inbox:triage`).

### Four Pillars

| Pillar | Emoji | Namespaces | Focus |
|--------|-------|---------|-------|
| Daily Work | 📧 | #01-#08 | Email, meetings, reviews |
| Code Without Coding | 🛠️ | #09-#16 | Reports, invoices, contracts |
| MCP & Integrations | 🔌 | #17-#24 | Notion, Drive, Slack, CRM |
| Meta & Growth | 📈 | #25-#32 | ROI, workflows, templates, memory, intelligence |

### Platform

All 32 namespaces run as a single plugin on **Claude Code**. The `"platform": "claude-code"` field in `.claude-plugin/plugin.json` confirms this.

### Agent Teams Patterns (Priority 5 namespaces)

| Pattern | Used By | How It Works |
|---------|---------|--------------|
| Pipeline | #01 Inbox Zero, #09 Report Gen | Sequential: Input → Agent A → Agent B → Output |
| Parallel Gathering | #02 Daily Briefing, #03 Meeting Prep, #20 Client Context | All agents fetch simultaneously, lead merges |
| Pipeline + Batch | #11 Invoice Processor | Pipeline per item, batch across items |
| Competing Hypotheses | #14 SOW Generator | Multiple agents propose, lead synthesizes |

### Autonomous Spectrum

Namespaces operate at different autonomy levels:

| Level | Trigger | Examples |
|-------|---------|---------|
| Interactive | User runs command | All 32 namespaces (default) |
| Scheduled | Cron/timer | P02, P05, P06, P10, P18, P19, P21, P22, P29 (via `--schedule`) |
| Workflow | P27 orchestration | Any namespace via `/founder-os:workflow:create` |

## MCP Servers & External Tools

The plugin uses these MCP servers and tools (install in priority order):

1. **Notion** MCP (21 namespaces) - CRM backbone, task tracking, output storage
2. **gws CLI** (20 namespaces) - Gmail, Calendar, and Drive access via `gws` CLI tool. Install once, authenticate with `gws auth login`. No per-namespace config needed.
3. **Filesystem** MCP (8 namespaces) - Local file processing, document generation
4. **Slack** MCP (2 namespaces) - Team communication digests
5. **Web Search** MCP (1 namespace) - Competitive research

### MCP Package Names
- Notion: `@modelcontextprotocol/server-notion`
- Filesystem: `@modelcontextprotocol/server-filesystem`
- Gmail/Calendar/Drive: **Use `gws` CLI** (not MCP servers). See `_infrastructure/gws-skills/`.

## Key Decisions

| Topic | Decision |
|-------|----------|
| CRM | Notion (not HubSpot/Pipedrive) |
| Voice transcription | Whisper (local, free, privacy-first) |
| Morning sync tools | Gmail + Calendar + Notion + Slack + Drive |
| Time savings tracking | Pre-defined task type estimates (not manual input) |
| Client health metrics | 5 scores 0-100: contact, response, tasks, payment, sentiment |

## Namespace Quick Reference (all 32 namespaces)

Each namespace's own files (SKILL.md, commands/, agents/) contain full implementation details. Read those when working with a specific namespace.

| # | Namespace | Command Prefix | Pattern | Required Tools | HQ DB |
|---|-----------|---------------|---------|---------------|-------|
| 01 | Inbox Zero | `/founder-os:inbox:` | Pipeline | gws (Gmail) | Tasks (Email Task), Content (Email Draft) |
| 02 | Daily Briefing | `/founder-os:briefing:` | Parallel Gathering | gws (Calendar, Gmail), Notion | Briefings (Daily Briefing) |
| 03 | Meeting Prep | `/founder-os:prep:` | Parallel Gathering | gws (Calendar, Gmail, Drive), Notion | Meetings |
| 04 | Action Items | `/founder-os:actions:` | None | Notion | Tasks (Action Item) |
| 05 | Weekly Review | `/founder-os:review:` | None | gws (Calendar, Gmail), Notion | Briefings (Weekly Review) |
| 06 | Follow-Up Tracker | `/founder-os:followup:` | None | gws (Gmail, Calendar), Notion | Tasks (Follow-Up) |
| 07 | Meeting Intel | `/founder-os:meeting:` | None | Notion, Filesystem | Meetings |
| 08 | Newsletter Engine | `/founder-os:newsletter:` | None | WebSearch, Filesystem | Content (Newsletter), Research (Newsletter Research) |
| 09 | Report Generator | `/founder-os:report:` | Pipeline | Filesystem, Notion | Reports (Business Report) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [thecloudtips/founder-os](https://github.com/thecloudtips/founder-os) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
