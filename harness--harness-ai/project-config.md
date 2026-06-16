---
trigger: always_on
description: Contribution guide for AI coding agents working in this repository. Follows the [agents.md](https://agents.md) specification. Human contributors should treat it as the canonical reference alongside [`README.md`](README.md).
---

# AGENTS.md

Contribution guide for AI coding agents working in this repository. Follows the [agents.md](https://agents.md) specification. Human contributors should treat it as the canonical reference alongside [`README.md`](README.md).

**Claude Code** reads `CLAUDE.md`, which is a symlink to this file.  
**Gemini CLI** reads `GEMINI.md` inside `extensions/gemini/`.  
**Cursor / Copilot / Codex / Aider** read this file directly.

## Project overview

`harness-ai` is a **distribution monorepo** that packages Harness AI integrations — skills, governance hooks, workspace rules, MCP config — as separate plugins/extensions for each major AI coding environment (Cursor, Claude Code, VS Code + Copilot, Gemini CLI). The Cursor plugin is published to the Cursor Marketplace; the others mirror the same skills surface for their respective platforms.

## Dev environment

```bash
# Required — Node 22+ for validator scripts (ESM)
node --version     # expect v22.x

# Clone and orient
git clone https://github.com/harness/harness-ai
cd harness-ai

# Skills mirror (upstream — edits happen there, not here)
git clone https://github.com/harness/harness-skills ../harness-skills
```

No `npm install` at repo root. Hook scripts under `plugins/cursor/scripts/` run with **Node stdlib only** (no dependencies).

## Build & test commands

File-scoped validators — run the closest one to what you changed.

```bash
# Everything (VS Code + Claude + Gemini + Cursor)
./scripts/validate.sh

# Cursor plugin deep check (manifest, mcp.json, hooks, frontmatter, skill-dir match)
( cd plugins/cursor && node scripts/validate-plugin.mjs )

# A single hook script — syntax only
node --check plugins/cursor/scripts/check-templates.mjs

# Smoke-test a hook fail-open path (should print {"permission":"allow"})
echo '{"tool_name":"MCP:harness_create","tool_input":{"resource_type":"pipeline","body":{}}}' \
  | env -u HARNESS_API_KEY -u HARNESS_ACCOUNT_ID ./plugins/cursor/scripts/check-templates.mjs

# Skills drift from upstream — must produce no diff after sync
./scripts/sync-skills.sh ../harness-skills
git diff --stat plugins/*/skills/
```

CI runs these on every PR touching `plugins/cursor/**` or `.cursor-plugin/marketplace.json` (`.github/workflows/cursor-plugin.yml`).

## Repository layout

```
harness-ai/
├── .cursor-plugin/marketplace.json   # Cursor multi-plugin manifest (lists one plugin)
├── plugins/
│   ├── cursor/                       # Harness Cursor plugin (marketplace-ready)
│   │   ├── .cursor-plugin/plugin.json
│   │   ├── mcp.json                  # Remote MCP (OAuth) default
│   │   ├── .mcp.local.json           # OSS/PAT sample — swap over mcp.json to use
│   │   ├── hooks/hooks.json          # governance hooks (templates + OPA)
│   │   ├── scripts/*.mjs             # hook implementations + validator
│   │   ├── rules/*.mdc               # workspace rules shipped with the plugin
│   │   ├── skills/<name>/SKILL.md    # mirrored from upstream
│   │   ├── assets/logo.svg
│   │   └── AGENTS.md                 # plugin-specific contribution rules
│   ├── claude/                       # Claude Code plugin (.claude-plugin/plugin.json)
│   └── vscode/                       # VS Code agent plugin (.github/plugin.json)
├── extensions/
│   └── gemini/                       # Gemini CLI extension (gemini-extension.json)
├── scripts/
│   ├── validate.sh                   # Umbrella validator
│   └── sync-skills.sh                # Mirror skills from upstream
└── .github/workflows/
    ├── cursor-plugin.yml             # PR validator for the Cursor plugin
    └── sync-skills.yml               # Daily auto-PR keeping skills trees in sync
```

## Per-plugin specs — follow the platform rules exactly

Each platform expects an exact manifest filename + location. Do **not** rename these.

| Plugin | Spec | Manifest | MCP config | Skills dir |
|--------|------|----------|------------|------------|
| Cursor | [cursor.com/docs/reference/plugins](https://cursor.com/docs/reference/plugins) | `plugins/cursor/.cursor-plugin/plugin.json` | `plugins/cursor/mcp.json` | `plugins/cursor/skills/` |
| Claude | [docs.claude.com/.../plugins-reference](https://docs.claude.com/en/docs/claude-code/plugins-reference) | `plugins/claude/.claude-plugin/plugin.json` | `plugins/claude/.mcp.json` | `plugins/claude/skills/` |
| VS Code | [code.visualstudio.com/.../agent-plugins](https://code.visualstudio.com/docs/copilot/customization/agent-plugins) (preview) | `plugins/vscode/.github/plugin.json` | `plugins/vscode/.mcp.json` | `plugins/vscode/skills/` |
| Gemini | [github.com/google-gemini/gemini-cli/tree/main/docs/extensions](https://github.com/google-gemini/gemini-cli/tree/main/docs/extensions) | `extensions/gemini/gemini-extension.json` | Inline in manifest | — |

When editing inside `plugins/cursor/`, the workspace rule [`plugins/cursor/rules/plugin-standards.mdc`](plugins/cursor/rules/plugin-standards.mdc) ships with the plugin and is also authoritative for Cursor-specific contribution details (hook event names, MCP matcher format, exact layout checks).

## Source of truth


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [harness/harness-ai](https://github.com/harness/harness-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
