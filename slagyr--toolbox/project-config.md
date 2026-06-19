---
trigger: always_on
description: Manages component dependencies (skills, commands, rules, modes, agents) for a project. Parses component URLs from the project's boot file, fetches them into a local .toolbox/ cache, projects them into agent-specific project-local paths (for example .claude/ and .opencode/), tracks freshness, and updates on demand. Use this skill when a project's boot file declares components via URL.
---


# Toolbox — Component Management

Resolve, cache, project, and update component dependencies declared in a project's boot file.

## When This Skill Applies

When you land in a project and the boot file contains a `## Toolbox` section with a link to this skill (toolbox), follow the procedure below to ensure all declared components are available locally before doing any work.

## Core Model: Cache + Projection

Toolbox has two layers:

1. **Canonical cache (`.toolbox/`)** for fetches, hashes, and update detection.
2. **Agent projection roots** for compatibility with agent-specific discovery paths.

Projection roots are always inside the repository (for example `.claude/` and `.opencode/`).

Never install or sync Toolbox-managed components to global locations such as `~/.claude` or `~/.config/opencode`.

## Agent Projection Paths (Project-Local Only)

Use these known project-local roots when applicable:

- **OpenCode:** `.opencode/`
- **Claude Code:** `.claude/`

Default relative layout under each projection root:

```
skills/{name}/SKILL.md
commands/{name}.md
rules/{name}.md
modes/{name}.md
agents/{name}.md
```

Projection selection rules:

1. Always project for the active agent.
2. Also project to other known roots that already exist in the repo.
3. Never project outside the repo root.
4. Prefer symlink; fallback to copy if symlinks are unavailable.

## Component Types

Toolbox manages different types of agent components. Each type has its own subsection under `## Toolbox` in the boot file, its own cache subdirectory, and its own entry in the manifest.

### Skills

Skills are instruction sets that teach agents how to perform specific tasks. A skill is a directory with a `SKILL.md` entry point and optional reference files.

- **Boot file section:** `### Skills` (under `## Toolbox`)
- **Cache location:** `.toolbox/skills/{name}/SKILL.md`
- **Projection location:** `{projection_root}/skills/{name}/SKILL.md`
- **Reference discovery:** Yes — relative markdown links in `SKILL.md` are fetched automatically.

### Commands

Commands are single-file agent instructions invoked by name (e.g., `/test`, `/deploy`). A command is a single markdown file.

- **Boot file section:** `### Commands` (under `## Toolbox`)
- **Cache location:** `.toolbox/commands/{name}.md`
- **Projection location:** `{projection_root}/commands/{name}.md`
- **Reference discovery:** No — commands are single files.

### Rules

Rules are behavioral constraints that modify how the agent operates (e.g., "always write tests", "never push to main"). A rule is a single markdown file.

- **Boot file section:** `### Rules` (under `## Toolbox`)
- **Cache location:** `.toolbox/rules/{name}.md`
- **Projection location:** `{projection_root}/rules/{name}.md`
- **Reference discovery:** No — rules are single files.

### Modes

Modes are operating profiles that configure the agent's behavior for a specific workflow (e.g., code-review, architect, planner). A mode is a single markdown file.

- **Boot file section:** `### Modes` (under `## Toolbox`)
- **Cache location:** `.toolbox/modes/{name}.md`
- **Projection location:** `{projection_root}/modes/{name}.md`
- **Reference discovery:** No — modes are single files.

### Agents

Agents are full persona definitions with system prompts and tool configurations. An agent is a single markdown file.

- **Boot file section:** `### Agents` (under `## Toolbox`)
- **Cache location:** `.toolbox/agents/{name}.md`
- **Projection location:** `{projection_root}/agents/{name}.md`
- **Reference discovery:** No — agents are single files.

## How Components Are Declared

Components are declared in the project's boot file under a `## Toolbox` section. The boot file is whatever file the agent reads on startup — `AGENTS.md`, `CLAUDE.md`, or any platform-specific equivalent. Each component is a markdown link in a bullet list under its type subsection:

```markdown
## Toolbox

This project uses [toolbox](https://raw.githubusercontent.com/slagyr/toolbox/main/SKILL.md)
to manage agent components. If `.toolbox/` doesn't exist, fetch the toolbox
SKILL.md from the URL above and follow its instructions. Once bootstrapped:

- **Skills:** Load from `.toolbox/skills/{name}/SKILL.md` when their descriptions match the task at hand.
- **Commands:** When the user invokes a command by name (e.g., "/test"), read and follow `.toolbox/commands/{name}.md`.
- **Rules:** Read and apply all rules from `.toolbox/rules/` at session start.
- **Modes:** When the user requests a mode by name, read and apply `.toolbox/modes/{name}.md`.
- **Agents:** When the user requests an agent by name, read and apply `.toolbox/agents/{name}.md`.
- **Agent Paths:** Project cached components into agent-local paths (for example `.claude/...` and `.opencode/...`) so each agent can discover them where it expects.

### Skills

- [tdd](https://raw.githubusercontent.com/slagyr/agent-lib/main/skills/tdd/SKILL.md)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [slagyr/toolbox](https://github.com/slagyr/toolbox) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-19 -->
