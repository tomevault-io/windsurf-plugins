---
trigger: always_on
description: > This file is for AI agents working on this codebase. It is a navigation map, not a knowledge dump — every section points at the source of truth instead of duplicating it.
---

# UniKit - Developer Guide

> This file is for AI agents working on this codebase. It is a navigation map, not a knowledge dump — every section points at the source of truth instead of duplicating it.

## What is UniKit?

`unikit-ai` is an npm CLI that bootstraps AI-agent context for game-dev projects. It supports **4 engines** (Unity, Godot 4, Godot 4 .NET, Unreal Engine 5) and **6 agents** (Claude Code, Codex CLI, Cursor, Gemini CLI, Qwen Code, OpenCode). It installs skills, subagents, MCP server configs, engine templates, and pulls knowledge-base rules from a remote registry.

Full prose: see `.ai-factory/DESCRIPTION.md`.

## Where to read what

This is the main navigation. Lists and details live in the source of truth — never duplicate them here.

| Need to know… | Source of truth | Type |
|---|---|---|
| Product overview, features, NFRs | `.ai-factory/DESCRIPTION.md` | prose |
| Layers, dependency rules, patterns | `.ai-factory/ARCHITECTURE.md` | prose |
| Deep conventions and operations | `CLAUDE.md` | prose |
| CLI contract (commands, exit codes) | `data/cli-contract.md` | machine-readable |
| Engine principles (with `{{vars}}`) | `data/dev-principles.md` | template |
| Skill list + descriptions | `skills/unikit-*/SKILL.md` (frontmatter) | filesystem |
| Subagent list + descriptions | `subagents/*.md` (frontmatter) | filesystem |
| Engine registry | `src/core/engines.ts` (`ENGINE_REGISTRY`) | code |
| Agent registry | `src/core/agents.ts` (`AGENT_REGISTRY`) | code |
| Transformer adapters | `src/core/transformers/` | code |
| MCP writers (per settings format) | `src/core/mcp-writers/` | code |
| Rules registry transports | `src/core/registry/` | code |
| MCP server templates | `mcp/{universal,unity,godot,unreal-engine-5}/*.json` | filesystem |
| Memory rules (core/stack per engine) | repo `NintendaDev/unikit-ai-rules` (+ `rules-registry/` snapshot) | external |
| Tests and validators | `scripts/test-*.sh` (entry: `test-skills.sh`) | bash |

## Project tree (top-level only)

```
unikit-ai/
├── bin/                  # CLI entry (single binary)
├── src/                  # TypeScript source — see ARCHITECTURE.md for layer rules
│   ├── cli/              #   presentation: commands + interactive wizard
│   ├── core/             #   business logic, including registry/ and mcp-writers/
│   └── utils/            #   leaf infra (fs-extra wrappers, hashing)
├── skills/               # Built-in skills (unikit-* prefix), copied to user projects
├── subagents/            # Built-in subagents (sidecars + coordinators)
├── mcp/                  # MCP server templates per engine + universal/
├── data/                 # CLI contract, dev principles, manifests, templates
├── rules-registry/       # Bundled snapshot of NintendaDev/unikit-ai-rules
│                         # (git-ignored, cloned at publish/CI/dev by scripts/download-rules.sh)
├── scripts/              # Bash test runners + download-rules.sh
└── dist/                 # Compiled JS output
```

## User project layout (`.unikit/`)

What `unikit-ai init` / `update` produces in the user's project root. Detailed write-paths live in `src/core/installer.ts`.

| Path | Owner command | Purpose |
|---|---|---|
| `.unikit.json` | `init` | Persistent config (agents, engine, `rulesRegistry`, `managedSkills`) |
| `.unikit/system/cli-contract.md` | `init`/`update` (flat-rewrite) | CLI contract for AI skills |
| `.unikit/system/dev-principles.md` | `init`/`update` (flat-rewrite) | Engine principles with `{{engine_*}}` substituted |
| `.unikit/memory/{core,stack}/*.md` | `rules install`/`sync` | Knowledge-base rules pulled from registry |
| `.unikit/memory/RULES_INDEX.md` | `rules sync` (regenerated) | Compact rules index — never hand-edit |
| `<agent-config>/skills/` | `init`/`update` | Installed skills (path varies per agent) |
| `<agent-config>/agents/` | `init`/`update` | Installed subagents |
| `.mcp.json` or per-agent settings | `init` | MCP server configs |

## Workflow at a glance

For per-skill details, read the corresponding `skills/unikit-<name>/SKILL.md`.

```
       Setup once                      Per feature
       ----------                      -----------
       /unikit                         /unikit-plan [fast|full|add]
          |                                   |
          v                                   v
       wizard installs               creates TASKS.md + PLAN-BRIEF.md
       skills + subagents +                   |
       MCP + system files                     v
          |                           /unikit-implement
          v                           (Bootstrap reads dev-principles.md
       /unikit Step 9                  once; executes with commit checkpoints)
       rules bootstrap                        |
       (`unikit-ai rules install`)            v
                                       /unikit-fix on bugs ->
                                       .unikit/patches/ -> /unikit-evolve

       Quality lane (read-only context):
       /unikit-commit  /unikit-review  /unikit-verify  /unikit-docs
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NintendaDev/unikit-ai](https://github.com/NintendaDev/unikit-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
