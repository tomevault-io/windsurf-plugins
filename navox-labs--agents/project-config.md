---
trigger: always_on
description: This is the build instruction file for the `navox-labs/agents` repository.
---

# CLAUDE.md — navox-labs/agents

This is the build instruction file for the `navox-labs/agents` repository.
Read this entire file before touching any file in this repo.

---

## What this repo is

A collection of Claude Code subagents — 8 specialist AI agents (7 engineers + 1 installer utility) that run entirely inside Claude Code sessions. No platform, no login, no data stored. Engineers install them globally or per project and hire them via slash commands.

This repo is open source under MIT. It lives at `github.com/navox-labs/agents`.

---

## Non-negotiable rules

- **Never modify agent prompt content without an explicit instruction to do so** — the prompts are the product
- **Never flatten the folder structure** — every file has a specific location for a reason
- **Never add dependencies** — this repo has zero dependencies by design
- **Never create binary files** — everything is markdown
- **Never rename agent files** — the filename becomes the slash command name
- **Always validate frontmatter** — malformed frontmatter breaks Claude Code agent loading
- **Always keep README.md in sync** — if you add/rename an agent, update the README table

---

## Commands

| Command | What it does |
|---|---|
| `/hire-team` | Onboard the full team, show handoff order |
| `/agency-run <task>` | Orchestrate the full team to complete a task end-to-end |
| `bash scripts/validate.sh` | Run repo integrity checks (111 checks across agents, docs, plugins, git) |

## Project memory

The team's institutional memory lives in two places:
- `.claude/project-memory.md` — shared across all agents, updated after every run
- `.claude/memory/[agent].md` — per-agent memory, updated after each agent run

Never delete these files. They are the team's knowledge base.

---

## Exact folder structure

This is the required structure. Do not deviate.

```
navox-labs/agents/
│
├── .gitignore
├── CLAUDE.md
├── README.md
├── GETTING-STARTED.md
├── LICENSE
│
├── .claude/
│   ├── agents/                        ← subagent definitions (one file per agent)
│   │   ├── architect.md               ← _architect (agent)
│   │   ├── devops.md                  ← _devops (agent)
│   │   ├── fullstack.md               ← _fullstack (agent)
│   │   ├── installer.md               ← _installer (agent, auto-dispatched — no command wrapper)
│   │   ├── local-review.md            ← local-review (agent, invoked by agency-run)
│   │   ├── qa.md                      ← _qa (agent)
│   │   ├── security.md                ← _security (agent)
│   │   └── ux.md                      ← _ux (agent)
│   │
│   ├── commands/                      ← slash commands
│   │   ├── agency-run.md              ← /agency-run (orchestrator)
│   │   ├── architect.md               ← /architect (command wrapper → _architect)
│   │   ├── devops.md                  ← /devops (command wrapper → _devops)
│   │   ├── fullstack.md               ← /fullstack (command wrapper → _fullstack)
│   │   ├── hire-team.md               ← /hire-team (onboarding)
│   │   ├── qa.md                      ← /qa (command wrapper → _qa)
│   │   ├── security.md               ← /security (command wrapper → _security)
│   │   └── ux.md                      ← /ux (command wrapper → _ux)
│   │
│   ├── memory/                        ← per-agent memory files (created at runtime)
│   │   └── [agent].md
│   │
│   ├── project-memory.md             ← shared project memory (created at runtime)
│   └── settings.local.json           ← local permission overrides (gitignored)
│
├── .claude-plugin/
│   ├── plugin.json                    ← plugin manifest for marketplace distribution
│   └── marketplace.json               ← marketplace registry
│
├── scripts/
│   └── validate.sh                    ← repo integrity checker (bash scripts/validate.sh)
│
├── templates/                         ← starter CLAUDE.md files per stack
│   ├── nextjs.CLAUDE.md
│   ├── node-api.CLAUDE.md
│   ├── rails.CLAUDE.md
│   ├── python-fastapi.CLAUDE.md
│   └── cloudflare-workers.CLAUDE.md
│
└── docs/
    ├── modes.md                       ← all modes for all agents
    ├── auth-ownership.md              ← auth responsibility table
    ├── handoff-chain.md               ← agent handoff flow diagram
    ├── hitl.md                        ← human-in-the-loop guide
    ├── parallel-execution.md          ← parallel agent execution guide
    └── install.md                     ← installation instructions
```

If a file or folder does not exist in this structure, create it.
If a file or folder exists that is not in this structure, ask before touching it.

---

## Agent file format (strict)

Every file in `.claude/agents/` must follow this exact format.
No exceptions. Malformed frontmatter silently breaks agent loading.

```markdown
---
name: agent-name-in-kebab-case
description: One sentence. What this agent does and when Claude should load it automatically. Include key trigger words.
---

[Full system prompt content here]
```

### Frontmatter rules

| Field | Rule |
|---|---|
| `name` | Lowercase. Agents use underscore prefix (`_architect`). Commands use plain name (`architect`). |
| `description` | One sentence. Used by Claude to auto-load the agent. Must include trigger keywords. |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [navox-labs/agents](https://github.com/navox-labs/agents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
