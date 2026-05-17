---
trigger: always_on
description: Gated pipeline orchestration for Claude Code and Codex CLI: specialist agents, slash commands, and skills.
---

# agentpipe

Gated pipeline orchestration for Claude Code and Codex CLI: specialist agents, slash commands, and skills.
9 agents, 15 commands, 2 skills (`gost-report`, `doc2kb`), 15 research docs. Multi-target installer:
default puts everything in `~/.claude/`; `--target codex` puts skills in `~/.agents/skills/`
(open-agent-skills standard) and skips agents/commands (Codex format differences).
The only build step is packaging skills into release zips — no runtime, no tests, no dependencies.

## Commands

```
bash install.sh                       # install for Claude Code (default)
bash install.sh --target codex        # install for Codex CLI (skills only)
bash install.sh --dry                 # preview what would change (works with --target too)
bash install.sh --diff                # show repo vs installed differences
bash install.sh --pull                # copy installed back to repo
bash install.sh --update              # git pull --ff-only, then install (alias of update.sh)
bash install.sh --uninstall           # remove installed files (target-scoped)
bash update.sh                        # canonical update entry point (forwards to install.sh --update)
bash install.sh --no-claude-md        # skip baseline CLAUDE.md (default: install-if-missing)
bash install.sh --skills-only         # update only skills/*, skip agents/commands/hooks (works with --target)
bash install.sh --with-sound-hooks    # opt-in: Stop sound hook only (one beep per turn)
bash install.sh --with-notification-sound  # opt-in: Notification sound hook only
bash install.sh --clean-sound-hooks   # strip all sound hooks (Stop+Notification) from settings.json
bash install.sh --with-thinking-summaries  # opt-in: showThinkingSummaries=true
bash install.sh --model-profile opus  # all agents on opus (default: mixed; persisted to settings.json)
bash scripts/build-skills.sh          # package every skills/<name>/ into dist/<name>.zip
bash scripts/eval.sh --list           # list local agent eval scenarios (no claude calls)
bash scripts/eval.sh <agent>          # run agent prompt-quality eval (uses claude -p, ~2 msgs/scenario)
```

## Structure

```
agents/*.md             Agent definitions — YAML frontmatter + system prompt
commands/*.md           Slash commands — YAML frontmatter + orchestration pipeline
skills/<name>/SKILL.md  Skills — folder with SKILL.md plus optional scripts/, references/
research/*.md           Reference docs, numbered 01-14. Not auto-imported by agents.
docs/                   User-facing documentation (commands.md, agents.md, installation.md, eval.md)
scripts/build-skills.*  Package skills/* into dist/*.zip for releases
scripts/eval.sh         Local prompt-quality eval runner (claude -p, no API key, no CI)
tests/<agent>/<scenario>/  Agent eval scenarios (input.md + rubric.md). Empty by default.
.github/workflows/      release.yml: on tag push, builds skill zips and attaches to GH release
install.sh              Bash installer (macOS, Linux, WSL, Git Bash)
install.ps1             PowerShell installer (Windows)
update.sh / update.ps1  Thin wrappers — forward to install with --update
scripts/CLAUDE.md.example  Neutral baseline CLAUDE.md (install-if-missing to ~/.claude/)
scripts/agentpipe.env.example  Curated reference for shell env vars (NOT auto-installed)
VERSION                 Semver string, read by installers and release workflow
CHANGELOG.md            Keep a Changelog format
```

## Agent Format (agents/*.md)

YAML frontmatter then markdown body. Exact fields:

```
name:         lowercase identifier (architect, dba, devops, docs, pm, refactorer, reviewer, security, tester)
description:  trigger sentence with "MUST BE USED when/before..." conditions
tools:        comma-separated allowlist from: Read, Write, Edit, Bash, Glob, Grep, WebFetch, WebSearch
model:        opus (for architect, security) or sonnet (all others)
```

Tool scoping by role:
- Read-only + web research: `Read, Grep, Glob, WebFetch, WebSearch` (architect, pm)
- Read + diagnostic bash: `Read, Grep, Glob, Bash` (dba, reviewer, security)
- Full write access: `Read, Write, Edit, Bash, Glob, Grep` (devops, tester, docs)

Body structure:
- `# [Role] Agent` — H1 title
- `## Core Responsibilities` — numbered list
- Domain-specific sections — rules, checklists, decision tables
- `## Output Format` or equivalent (e.g. `## Specification Format`) — template the agent must follow
- `## Handoff Protocol` — `## Next Steps` with `RECOMMEND:` lines to other agents

## Command Format (commands/*.md)

YAML frontmatter then markdown body. Exact fields:

```
description:    quoted string, one sentence explaining the command
argument-hint:  (optional) placeholder shown to user, e.g. <feature description>
```

Body structure:
- `## Context` containing `@CLAUDE.md` — always present, loads project instructions
- `$ARGUMENTS` under a descriptive heading — present when argument-hint exists
- `## Pipeline` with `### Step N: Name` — sequential, gated steps (most commands)
- Loop-style commands (e.g. sprint) use `## Loop` with `### Phase N:` instead
- Agent invocation: `Run the [name] agent:` followed by prompt in quotes

Commands without argument-hint: next, onboard, deploy.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zevtos/agentpipe](https://github.com/zevtos/agentpipe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
