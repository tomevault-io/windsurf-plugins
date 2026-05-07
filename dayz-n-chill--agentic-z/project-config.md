---
trigger: always_on
description: This file provides foundational guidance for agents working in this repository.
---

# Repository Guidelines

This file provides foundational guidance for agents working in this repository.

## Purpose

This repo is **Agentic-Z** — an AI Agent Stack for DayZ Modding, with first-class support for Claude Code, Codex, and Gemini. Cloning it starts a new DayZ mod project. Every clone inherits the same scaffolding: rules, agents, skills, helper scripts.

## Default rules (apply to every clone)

These are the L1 rules of the template. They apply to every clone of this repo, every agent, every skill. DayZ-specific rules live separately at `.claude/skills/_shared/dayz-conventions.md` and are referenced from the individual agent/skill files that need them.

**Skim summary:**

1. **Communication** — answer first, caveat after.
2. **Tooling** — dedicated tools (`Read`/`Edit`/`Write`/`Glob`/`Grep`) first; Python for non-trivial; PowerShell only when faster.
3. **Memory** — `.claude/local-memory/` for user/machine notes; never for rules.
4. **Subagent routing**. For DayZ work, dispatch the matching `dayz-*` specialist instead of generic Superpowers agents. Role-to-specialist table in `docs/dayz-modding.md`.

### Communication — answer first, caveat after

When asked for something an agent can't literally do (read a UI pane, see the screen, access a closed system), infer what the user actually wants and deliver it via available tools. Mention the limitation as a one-liner *after* the answer, never as the lead. Example: "Here's the markdownlint output your Problems pane is showing — note I can't read the pane directly, but the underlying linter is the same." Not: "I can't read the Problems pane."

### Tooling — pick the fastest tool for the job

- **Dedicated tools first** — `Read`, `Edit`, `Write`, `Glob`, `Grep`. They run in-process (no cold start), return structured output (line-numbered Reads, sorted Globs, output-mode Greps), and handle Windows paths without quoting drama. They replace shell-side `cat`/`head`/`tail`, `sed`/regex-replace, `echo > file`, `find -name`/`ls -R`, and `grep -r`/`rg` respectively. Use them whenever they fit. They do **not** replace process-invocation tools (`git`, `npm`, `taskkill`, `subst`) — those still go through Bash/PowerShell.
- **Default to Python** for non-trivial work that doesn't fit a dedicated tool — parsing, loops, JSON/XML, file ops with logic, anything multi-step. ~150ms cold start.
- **cmd `.bat`** is fine for trivial Windows-native wrappers (calling an `.exe`, junctions, taskkill, etc.).
- **PowerShell** is allowed, but only when (a) you explicitly ask for it, or (b) it's genuinely faster than the alternatives for the task. Avoid as a default — PowerShell's cold-start (~1.5s) and folder traversal are slow compared to Python.
- **Bash** for trivial one-liners only; cold-start is similar to PowerShell on Windows.

### Memory — `.claude/local-memory/` only, never for rules

User/machine-specific notes go in `<repo>/.claude/local-memory/` (gitignored, per-clone). Never store rules or conventions there — those go in the repo so they travel with every clone. L1 rules live in this file; L2 DayZ rules live at `.claude/skills/_shared/dayz-conventions.md`.

### Subagent routing for DayZ work

When a Superpowers workflow (`/brainstorming`, `/executing-plans`, `/requesting-code-review`, `/dispatching-parallel-agents`, etc.) needs to dispatch a subagent and the work touches DayZ, route to the matching `dayz-*` specialist via the Agent tool. Generic Superpowers agents (e.g. `superpowers:code-reviewer`) don't know Enforce Script conventions and will miss DayZ-specific traps like the `modded class` no-extends rule or engine classes that can't be modded.

Heuristic: anything under `workspace/<ModName>/`, `.claude/skills/dayz-*/`, `P:\`, or DayZ file types (`.c`, `.layout`, `.p3d`, `.paa`, `.rvmat`, `config.cpp`, `types.xml`) is DayZ work. See the role-to-specialist table at `docs/dayz-modding.md` ("Superpowers ↔ DayZ specialist routing").

For non-DayZ work (Python skills, scripts, infra, the wiki), Superpowers' generic agents are fine.

## Repository Use

- **Bootstrap**: After cloning, run `/sync-skills` (or `python .claude/skills/sync-skills/sync.py`). This links the repo's `.claude/skills/` into Claude Code, Codex, and Gemini home dirs so all three agents discover the same slash commands. The agent list lives in `.claude/skills/sync-skills/agents.json` — adding a new agent CLI later is a one-entry edit.
- Agent definitions live in `.claude/agents/`.
- Agent memory lives in `.claude/agent-memory/` (per-user, gitignored; auto-created on first write).
- Reusable skills live in `.claude/skills/`.
- Local settings live in `.claude/settings.local.json` (per-user, gitignored). Project-wide settings go in `.claude/settings.json`.
- Keep the repository root clean.
- `output/` is the **default destination for one-shot deliverables** — documents, decks, generated files, and other artifacts the user takes elsewhere. Goes under `output/<descriptive-folder>/` unless the user names a different destination.
- `workspace/` is for **in-progress mod projects** — anything you're actively iterating on across sessions. Each mod gets its own subfolder (`workspace/<ModName>/`).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DayZ-n-Chill/Agentic-Z](https://github.com/DayZ-n-Chill/Agentic-Z) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
