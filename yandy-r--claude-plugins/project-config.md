---
trigger: always_on
description: This repository ships a single Claude Code plugin called `ycc`, distributed via the
---

# Claude Plugins - Project Instructions

## Overview

This repository ships a single Claude Code plugin called `ycc`, distributed via the
marketplace at `.claude-plugin/marketplace.json`. All skills, commands, and agents live
under `ycc/` and are accessed at runtime as `ycc:{skill}`, `/ycc:{command}`, or
`subagent_type: "ycc:{agent}"`.

The same source trees also generate native compatibility bundles for Cursor, Codex, and
opencode:

- Cursor bundle: `.cursor-plugin/`
- Codex bundle: `.codex-plugin/ycc/`
- Codex custom agents: `.codex-plugin/agents/`
- opencode bundle: `.opencode-plugin/` (skills, agents, commands, AGENTS.md, opencode.json)

> Pre-2.0 versions shipped 9 separate plugins. 2.0.0 collapsed them into one bundle so
> every skill is reachable via the same `ycc:` namespace prefix. See `docs/plans/` for
> the consolidation plan.

## Repository Layout

```
claude-plugins/
├── .claude-plugin/
│   └── marketplace.json     # single ycc entry
├── .codex-plugin/
│   ├── agents/              # generated Codex custom agents
│   └── ycc/                 # generated Codex plugin root
├── .cursor-plugin/          # generated Cursor bundle
├── .opencode-plugin/        # generated opencode bundle
│   ├── skills/              # → ~/.config/opencode/skills/
│   ├── agents/              # → ~/.config/opencode/agents/
│   ├── commands/            # → ~/.config/opencode/commands/
│   ├── AGENTS.md            # opencode rules file
│   └── opencode.json        # schema + default model + MCP translation
├── ycc/
│   ├── .claude-plugin/
│   │   └── plugin.json      # name: "ycc", version: 2.0.0
│   ├── commands/            # slash commands (one .md per command)
│   ├── agents/              # agents (one .md per agent)
│   └── skills/
│       ├── _shared/         # cross-skill helper scripts
│       └── {skill-name}/
│           ├── SKILL.md     # skill prompt (required)
│           ├── references/  # templates, examples, reference docs
│           └── scripts/     # validation and helper scripts
└── docs/
    └── plans/               # implementation plans
```

## Plugin Development Conventions

### Naming

- The plugin name is `ycc` and must NOT change. The marketplace prefix is always `ycc:`.
- Skills: `kebab-case` directory under `ycc/skills/`. The directory name becomes the
  skill identifier (e.g., `ycc/skills/git-workflow/` → `ycc:git-workflow`).
- Commands: `kebab-case.md` under `ycc/commands/`. The basename becomes the slash
  command (e.g., `ycc/commands/clean.md` → `/ycc:clean`).
- Agents: `kebab-case.md` under `ycc/agents/`. The basename becomes the agent identifier
  (e.g., `ycc/agents/codebase-advisor.md` → `subagent_type: "ycc:codebase-advisor"`).
- Scripts: `kebab-case.sh` with bash shebang.

### Scripts

All scripts must:

- Start with `#!/usr/bin/env bash`
- Use `set -euo pipefail` for safety
- Include validation guards (check required inputs exist)
- Exit with meaningful codes (0 = success, 1 = error)
- Write output to stdout, errors to stderr

When a script needs to reference its own plugin path, use `${CLAUDE_PLUGIN_ROOT}` —
this resolves to the `ycc/` directory at runtime. Paths inside skills follow the form
`${CLAUDE_PLUGIN_ROOT}/skills/{skill-name}/...`.

Codex-generated skills are not source-edited directly. The Codex generator rewrites
those paths to the managed install location `~/.codex/plugins/ycc/...`.

### Skills

Each skill directory contains:

- `SKILL.md` — the skill prompt (required)
- `references/` — templates, examples, and reference docs
- `scripts/` — validation and helper scripts

### Cross-skill helpers

Shared helpers (used by more than one skill) live under `ycc/skills/_shared/scripts/`.
Skills source them via `${CLAUDE_PLUGIN_ROOT}/skills/_shared/scripts/{name}.sh`.

### Registration

The marketplace registry at `.claude-plugin/marketplace.json` contains a single entry:

```json
{
  "name": "ycc",
  "version": "2.0.0",
  "source": "./ycc"
}
```

Do not add additional plugin entries. New functionality goes into the existing `ycc`
plugin as a new skill, command, or agent. See `CONTRIBUTING.md` → Scope & Guardrails for
the full policy on what belongs in `ycc` and what to reject.

## Generated Compatibility Targets

- `ycc/skills/` is the source of truth for Cursor, Codex, and opencode skill generation.
- `ycc/agents/` is the source of truth for Cursor, Codex, and opencode agent generation.
- `ycc/commands/` is the source of truth for opencode command generation (Cursor does not
  natively consume `.md` commands; Codex has no slash-command layer).
- Do not hand-edit generated files under `.cursor-plugin/`, `.codex-plugin/`, or
  `.opencode-plugin/` unless you are first changing the generator.
- Codex does not support this repo's custom slash-command layer as installable artifacts.
  The native Codex target exposes skills via the plugin bundle and agents via TOML files.
- opencode has first-class support for skills, agents, AND commands. The opencode bundle
  ships all three, plus an `opencode.json` config (with MCP translated from
  `mcp-configs/mcp.json`) and an `AGENTS.md` rules file derived from this document.
- See [`ycc/skills/_shared/references/target-capability-matrix.md`](ycc/skills/_shared/references/target-capability-matrix.md)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yandy-r/claude-plugins](https://github.com/yandy-r/claude-plugins) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-19 -->
