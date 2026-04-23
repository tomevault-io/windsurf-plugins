---
trigger: always_on
description: <!-- Quick start: In Claude Code, read @AGENTS.md to run bootstrap, session checks, and task routing -->
---

<!-- Quick start: In Claude Code, read @AGENTS.md to run bootstrap, session checks, and task routing -->

> **Claude Code / Codex / other agents -- determine your context before proceeding:**
>
> **Source repo test:** If `bootstrap/bootstrap.sh`, `bootstrap/bootstrap.ps1`, and `skills/` all exist at the repo root, you are in the `anywhere-agents` source repo (or a fork with the same layout). **Skip the bootstrap block below** and proceed directly to `## Session Start Check`. Do not execute the bootstrap script in the source repo; it would try to bootstrap itself.
>
> **Consumer repo path:** Otherwise, you are in a project that consumes the shared config. **You MUST execute the matching bootstrap block below at the start of every session, before responding to any user request or running session checks.** Use the PowerShell block on Windows and the Bash block on macOS/Linux. Bootstrap is idempotent: run it every session even if `.agent-config/` already exists, so shared config, skills, and settings are refreshed from upstream before any work begins. Do not treat the block below as reference documentation -- it is a required runtime command. After running, report the refresh result in one line (for example, "bootstrap: refreshed AGENTS.md, skills, settings").

## Bootstrap: Shared Config and Skills

This file is the shared agent configuration hosted at `https://github.com/yzhao062/anywhere-agents`.
Project repos should include the following bootstrap block in their own `AGENTS.md` so that agents automatically pull the latest shared defaults and skills before starting any task. Because bootstrap overwrites the consumer's root `AGENTS.md` with this source on every run, the block below ends up serving two roles in consumer repos: it is both the embedded reference for how the setup works and the runtime command the agent executes each session (see the top-of-file note for the source-vs-consumer test that decides which role applies in the current repo).

### Bootstrap block for project repos

Copy the block below into the top of each project repo's `AGENTS.md`. Because bootstrap overwrites `AGENTS.md` on every run, put project-local overrides in `AGENTS.local.md`.

````
## Shared Agent Config (auto-fetched)

Before starting any task, fetch the shared agent configuration and skills. Use the block for your shell.

PowerShell (Windows):

```powershell
New-Item -ItemType Directory -Force -Path .agent-config, .claude, .claude/commands | Out-Null
Invoke-WebRequest -UseBasicParsing -Uri https://raw.githubusercontent.com/yzhao062/anywhere-agents/main/bootstrap/bootstrap.ps1 -OutFile .agent-config/bootstrap.ps1
& .\.agent-config\bootstrap.ps1
```

Bash (macOS/Linux):

```bash
mkdir -p .agent-config .claude/commands
curl -sfL https://raw.githubusercontent.com/yzhao062/anywhere-agents/main/bootstrap/bootstrap.sh -o .agent-config/bootstrap.sh
bash .agent-config/bootstrap.sh
```

This bootstrap flow refreshes the consuming repo's root `AGENTS.md` to match the shared copy. If a project later needs repo-local overrides, put them in `AGENTS.local.md`.

Read and follow the rules in `.agent-config/AGENTS.md` as baseline defaults. Any rule in `AGENTS.local.md` overrides the shared default.
When a skill is invoked, read its SKILL.md from `.agent-config/repo/skills/<skill-name>/SKILL.md`.
If a local `skills/<skill-name>/SKILL.md` exists in the project repo, the local copy takes precedence.
Copying `.agent-config/repo/.claude/commands/*.md` only overwrites command files with the same name as the shared repo and does not delete unrelated project-local commands.
Merge shared Claude project defaults (e.g., `permissions`, `attribution`) from `.agent-config/repo/.claude/settings.json` into the project `.claude/settings.json`. Shared keys are updated on every bootstrap run; project-only keys are preserved. Merge requires Python; if unavailable the existing file is left untouched.
Add `.agent-config/` to the project's `.gitignore` so fetched files are not committed.
Bootstrap also sets up user-level config: it copies `scripts/guard.py` to `~/.claude/hooks/` (a PreToolUse hook that guards against destructive commands) and merges `user/settings.json` into `~/.claude/settings.json` (shared permissions, hook wiring, and the `CLAUDE_CODE_EFFORT_LEVEL=max` env entry that sets the default effort level). Remove the user-level section from the bootstrap script if this is not wanted.
````

### What gets shared

| Content | Source | How fetched |
|---------|--------|-------------|
| User profile, writing defaults, formatting rules, environment notes | `AGENTS.md` (this file) | `curl` raw file |
| Per-agent rule files (`CLAUDE.md`, `agents/codex.md`) | Generated from `AGENTS.md` by `scripts/generate_agent_configs.py` | Regenerated locally on every bootstrap; hand-authored files preserved + warned |
| Shared skills (`implement-review`, `my-router`, `ci-mockup-figure`, `readme-polish`) | `skills/` directory (committed only) | sparse `git clone` |
| Claude pointer commands for shared skills | `.claude/commands/` | sparse `git clone` plus non-destructive copy into the project `.claude/commands/` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yzhao062/anywhere-agents](https://github.com/yzhao062/anywhere-agents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
