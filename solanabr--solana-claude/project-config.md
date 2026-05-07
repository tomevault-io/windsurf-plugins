---
trigger: always_on
description: <!-- This is the config-repo maintainer file (NOT shipped to user projects).
---

# Solana Claude Config - Meta Configuration
<!-- This is the config-repo maintainer file (NOT shipped to user projects).
     CLAUDE-solana.md is the one that ships as CLAUDE.md to target projects. -->

This repository contains Claude Code configuration for Solana development projects. The actual Solana builder configuration lives in `CLAUDE-solana.md` and should be copied to target projects as their `CLAUDE.md`.

**Install docs**: See README.md and QUICK-START.md.

---

## This Repo's Purpose

You are maintaining the **solana-claude-config** repository - a template/library of Claude Code configurations for Solana development. Your role is to improve, test, and maintain the agents, skills, commands, MCP servers, and rules that other projects will use.

## Token Loading Model
<!-- WHY: Understanding when each file loads determines your token budget.
     CLAUDE.md is a user message (not system prompt) — shorter = better adherence.
     Rules without paths: load at session start, so keep them minimal. -->

| File | When loaded | Budget guidance |
|------|-------------|-----------------|
| `CLAUDE.md` | Session start; delivered as user message (uncached) | Keep <200 lines; costs every turn |
| `CLAUDE-solana.md` | Session start (user projects) | Keep <120 lines; uncached; HTML comments stripped (free) |
| `MEMORY.md` | Session start | 200-line / 25KB cap; index pointers only |
| `.claude/rules/*.md` (with `paths:`) | Lazy — on matching file read | Can be detailed; zero startup cost |
| `.claude/rules/*.md` (no `paths:`) | Session start | Minimal — always loaded |
| `.claude/agents/*.md` | On agent spawn | Can be detailed |
| `.claude/commands/*.md` | On invocation | Can be detailed |
| `.claude/skills/SKILL.md` | On invocation | Medium; HTML comments NOT stripped |
| `.claude/skills/*.md` | On-demand via links | Can be detailed |
| Subdirectory `CLAUDE.md` | Lazy — when Claude reads files in that dir | Monorepo module configs |

## Communication Style

- No filler phrases ("I get it", "Awesome, here's what I'll do", "Great question")
- Direct, efficient responses — code/config first, explanations when needed
- Admit uncertainty rather than guess
- Consider token efficiency in all additions

## Common Mistakes

**DON'T**:
- Edit CLAUDE-solana.md without considering it ships to user projects (different audience than this repo)
- Add agent/skill content that duplicates what's already in external submodules
- Reference files by line number in CLAUDE.md — line numbers shift constantly
- Forget .env.example when adding/removing MCP servers
- Leave stale counts (e.g., "15 agents", "22 commands") — grep to verify before committing

**DO**:
- Run `bash validate.sh && bash tests/run_all.sh` before every commit
- Check QUICK-START.md and README.md after any structural change
- Test install.sh in a temp dir after modifying it
- Keep CLAUDE-solana.md under 120 lines — it loads on every user conversation

## Ripple Map
<!-- CRITICAL: This is the #1 cause of stale docs. When adding/removing
     any component, walk through every row before committing. -->

When X changes, also update Y:

| Changed | Also update |
|---------|-------------|
| Add/remove **agent** | README.md agent table + tree count, QUICK-START.md tree count, install.sh output, tests/test_agents.sh + test_install.sh assertions |
| Add/remove **command** | README.md commands tables + tree count, QUICK-START.md tree count, tests/test_commands.sh + test_install.sh assertions |
| Add/remove **MCP server** | README.md MCP table, CLAUDE-solana.md MCP list, QUICK-START.md MCP list, .env.example, .claude/commands/setup-mcp.md |
| Add/remove **submodule** | .gitmodules, README.md submodules table + tree, QUICK-START.md tree, .claude/skills/SKILL.md routing |
| Modify **install.sh** | Test: `bash tests/test_install.sh` in temp dir |
| Modify **CLAUDE-solana.md** | This ships to ALL user projects — different audience than this repo |

## Submodule Pitfalls

- **Never** `git add .claude/skills/ext/<dir>` — commits as tree, not submodule. Use `git submodule add <url> .claude/skills/ext/<name>` then `git add .gitmodules .claude/skills/ext/<name>`.
- Path renames in upstream submodules ripple into all agents + commands that reference skill files. Grep for old path before committing.
- install.sh silently skips submodule init if target isn't a git repo — intentional, not a bug.

## When Editing This Repo

| Component | Location | Key Rule |
|-----------|----------|----------|
| **Agents** | `.claude/agents/` | Non-overlapping responsibilities; spawn other agents for cross-domain work |
| **Skills** | `.claude/skills/` | Progressive loading; reference from `SKILL.md`; prefer code over prose |
| **Commands** | `.claude/commands/` | Atomic (one command, one purpose); document inputs/outputs |
| **Rules** | `.claude/rules/` | Minimal — they load on every matching file; use `globs` in frontmatter |
| **MCP Servers** | `.mcp.json` | Document env vars; test connectivity; update setup-mcp command |

## Agent Teams

Teams are dynamic — created via natural language, not static config (`CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS` enabled in settings.json). See README.md for recommended team patterns.

## Branch Workflow


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [solanabr/solana-claude](https://github.com/solanabr/solana-claude) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
