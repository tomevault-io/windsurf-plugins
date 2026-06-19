---
trigger: always_on
description: Use when creating or updating a pure-Markdown orientation layer: lean ORIENT.md maps, thin README/AGENTS.md/CLAUDE.md pointers, Obsidian MOCs, and parallel read-only subtree exploration reports. No scripts or generated code required.
---


# Orient

## Overview

Create a lean, text-only orientation layer for a repository, documentation tree, or Obsidian-style vault so future agents do less blind discovery before they act.

The canonical file is `ORIENT.md`: a compact Markdown map of where things live, what to read first, what commands matter, and what areas are risky or off-limits. Agent-facing files such as `README.md`, `AGENTS.md`, and `CLAUDE.md` should stay thin and point readers to `ORIENT.md`.

Core principle: **map first, details on demand**. Do not dump the repo into instructions. Give routing hints, entrypoints, ownership boundaries, and search/read recipes.

Hard constraint: **do not create scripts, Python files, generated CLIs, or non-Markdown helper code for this workflow unless the user explicitly asks.** This skill is instructions-only and uses normal agent tools: search, read, write, patch, and delegation.

## When to Use

Use this skill when the user asks to:
- Set up `ORIENT.md`, `README.md` map pointers, `AGENTS.md`, `CLAUDE.md`, MOCs, repo maps, or agent-facing context files.
- Reduce agent time spent exploring a large codebase or vault.
- Create cross-agent orientation files for Claude Code, Codex, Cursor, Gemini, Hermes, OpenCode, Copilot, Pi, Factory/Droid, or generic agents (AGENTS-compatible non-Claude tools are routed through `AGENTS.md`).
- Build a "where to look for what" map for a monorepo, docs site, knowledge base, or Obsidian vault.
- Parallelize repository discovery using read-only subagents and synthesize their reports.
- Rebrand older `orient-map` or `yourient` conventions to `orient` + `ORIENT.md`.

Do not use for full implementation plans, detailed architecture docs, generated API references, or exhaustive code packing. Use external repo-map tools only if the user asks.

## Branding Rules

### Slug Source

The callable slug comes from `SKILL.md` frontmatter `name:`, not the directory name. If you rename the folder, also rename the frontmatter — and vice versa. Two skill folders with the same frontmatter `name:` make agent skill lookup ambiguous; keep one canonical copy per ecosystem.

### Names

Use this branding consistently:

- Skill / slash-command name: `orient`
- Canonical file name: `ORIENT.md`
- Nested map files: `<subtree>/ORIENT.md`
- Adapter block markers: `<!-- ORIENT:BEGIN v=1 -->` / `<!-- ORIENT:END v=1 -->`
- README block markers: `<!-- ORIENT-README:BEGIN v=1 -->` / `<!-- ORIENT-README:END v=1 -->`

Do **not** emit `yourient`, `YOURIENT`, or `orient-map` as active branding.

Reference: `references/rebrand-and-pointer-blocks.md` contains the exact full-rebrand checklist plus the managed README and adapter block shapes.

## Naming Standard

Default to this hierarchy:

```text
ORIENT.md                  # canonical portable orientation map
README.md                   # short human pointer to ORIENT.md
AGENTS.md                   # generic adapter — Codex, Cursor, Gemini, Copilot, OpenCode, Hermes, ...
CLAUDE.md                   # Claude Code adapter
<subtree>/ORIENT.md        # optional local orientation for large subsystems
<subtree>/AGENTS.md         # optional local adapter when nearest-file precedence helps
```

Use uppercase `ORIENT.md` consistently. If a project already has `orient.md`, `orientation.md`, or pre-1.5 map files, follow the migration procedure in `references/rebrand-and-pointer-blocks.md`.

## Agent Adapter Targets

The cross-tool baseline is [`AGENTS.md`](https://agents.md) — an open convention supported by a growing set of coding agents and tools, including OpenAI Codex, Cursor, GitHub Copilot coding agent, Gemini CLI, Aider, OpenCode, Windsurf, Zed, Factory, and others. Orient layers atop it: `ORIENT.md` holds the canonical map; `AGENTS.md` and the Claude-specific `CLAUDE.md` point to that map. The skill does not replace `AGENTS.md`; it gives it something concrete to link to. By default orient emits only these two adapters — `AGENTS.md` for AGENTS-compatible tools and `CLAUDE.md` for Claude Code — and does not create `GEMINI.md`, `.cursor/rules/*.mdc`, or other tool-specific files unless the user explicitly asks.

Ask the user which agents to target if they did not specify. If they do not answer, default to portable files only: `ORIENT.md` + `README.md` + `AGENTS.md`.

| Target | Project file(s) | Notes |
|---|---|---|
| Humans / contributors | `README.md` | Add a short "read `ORIENT.md` first" pointer, not the full map. |
| Generic / AGENTS-compatible | `AGENTS.md` | Widest portable adapter. Keep short. |
| OpenAI Codex | `AGENTS.md` | Supports nested `AGENTS.md`; nearest file generally wins. |
| Hermes | `AGENTS.md` | Use as project context when supported; keep canonical guidance in `ORIENT.md`. |
| Claude Code | `CLAUDE.md` | Claude Code project memory. Add a short orientation block pointing to `ORIENT.md`. |
| Gemini CLI / Antigravity | `AGENTS.md` | Gemini CLI can be configured to use `AGENTS.md`; do not emit `GEMINI.md` unless the user explicitly asks. |
| Cursor | `AGENTS.md` | Supported by the AGENTS.md convention. Keep the adapter short. |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jqbit/orient](https://github.com/jqbit/orient) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
