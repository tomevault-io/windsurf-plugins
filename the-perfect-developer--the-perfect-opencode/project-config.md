---
trigger: always_on
description: A curated collection of agents, skills, and commands for [OpenCode](https://opencode.ai).
---

# OpenCode Base Collection — Agent Instructions

A curated collection of agents, skills, and commands for [OpenCode](https://opencode.ai).
This is a **content/configuration repository** — no application source code to compile or run.
All tools live under `.opencode/` (agents, skills, commands) and are written in Markdown.

## Project Structure

```
.opencode/
├── agents/        # Agent definition files (.md with YAML frontmatter)
├── commands/      # Slash command definitions (.md with YAML frontmatter)
├── skills/        # 35+ skill directories, each containing SKILL.md
├── package.json   # Bun package: @opencode-ai/plugin
└── bun.lock
.githooks/         # Version-controlled git hooks
├── pre-commit     # Orchestrator — runs all hooks in hooks.d/
└── hooks.d/       # Numbered hook scripts (10-, 20-, 30-, 40-)
scripts/           # Bash automation (install, setup, catalog generation)
pyproject.toml     # Ruff linter/formatter config (Python tooling only)
```

## Validation Commands

There is no build step. Validation is the primary quality gate.

```bash
# Validate all bash scripts (syntax only — does not execute)
bash -n scripts/install.sh
bash -n scripts/setup-hooks.sh

# Validate a specific skill
.opencode/skills/skill-creation/scripts/validate-skill.sh .opencode/skills/<skill-name>

# Run all pre-commit validations manually
.githooks/pre-commit

# Individual hook scripts
.githooks/hooks.d/10-validate-bash.sh      # bash -n on staged .sh files
.githooks/hooks.d/20-validate-skills.sh    # SKILL.md frontmatter validation
.githooks/hooks.d/30-validate-python-ruff.sh  # ruff lint + format check
.githooks/hooks.d/40-validate-eslint.sh    # ESLint on JS/TS files

# Python linting (Ruff)
ruff check .               # lint
ruff check --fix .         # lint + auto-fix
ruff format .              # format
ruff format --check .      # format check (CI mode, no writes)
```

### Set Up Git Hooks (first-time)

```bash
./scripts/setup-hooks.sh   # sets git core.hooksPath = .githooks
```

The pre-commit orchestrator runs 4 hooks in sequence:
- `10-validate-bash.sh` — bash `-n` syntax check on staged `.sh` files
- `20-validate-skills.sh` — SKILL.md frontmatter validation
- `30-validate-python-ruff.sh` — Ruff lint + format check
- `40-validate-eslint.sh` — ESLint on JS/TS files (activates only in consumer projects with a root `package.json`)

## Skill Validation Rules

A SKILL.md is valid when:
- Frontmatter delimiters `---` are present and well-formed
- `name` field: `^[a-z0-9]+(-[a-z0-9]+)*$`, 1–64 chars, matches directory name
- `description` field: 1–1024 chars, should include quoted trigger phrases
- Body: 1500–2000 words ideal; move excess to `references/`
- Scripts in `scripts/` must be executable (`chmod +x`)

## Code Style Guidelines

### Bash Scripts

- Shebang: `#!/usr/bin/env bash` preferred for portability; `#!/bin/bash` also acceptable (used in hook scripts)
- Primary install script (`scripts/install.sh`) uses `#!/usr/bin/env bash`; hook scripts use `#!/bin/bash` for historical reasons; prefer `#!/usr/bin/env bash` in new scripts
- Use `set -uo pipefail` (not `set -e`; avoid silent failures). Note: existing hook scripts use `set -e` for historical reasons; new scripts should use `set -uo pipefail`
- Make scripts executable: `chmod +x`
- Validate with `bash -n` before committing
- Name hook scripts with numeric prefix: `<NN>-<description>.sh` (increments of 10)
- Exit early if no relevant staged files (fast-fail pattern)
- Use ANSI color constants (`RED`, `GREEN`, `YELLOW`, `NC`) for output
- Use `echo -e` with color variables for status messages
- Heredocs for multi-line usage text (`cat <<EOF ... EOF`)

### Markdown / SKILL.md Files

- YAML frontmatter block at the top between `---` delimiters
- Required frontmatter fields: `name`, `description`
- Optional frontmatter fields: `license`, `compatibility`, `metadata`
- `name` must be lowercase-hyphenated and match the parent directory name
- `description` must include quoted trigger phrases (e.g., `"create a skill"`)
- Use ATX headings (`#`, `##`, `###`), not Setext style
- Fenced code blocks with language tag (` ```bash `, ` ```yaml `, etc.)
- Progressive disclosure: keep SKILL.md body focused; put deep-dives in `references/`

### Agent Files (`.opencode/agents/*.md`)

Agent `.md` files contain **only** `description` and `mode` in their YAML frontmatter. All tool access and permissions are defined in `opencode.json` under `agent.<name>.permission` — never in the `.md` file.

```yaml
---
description: "..."
mode: subagent
---
```

#### Agent Roles

| Role | Agents | `permission.write` | `permission.edit` | Can commit? |
|---|---|---|---|---|
| Orchestrator | orchestrix | `deny` | `deny` | No |
| Consultant | principal-architect, solution-architect, database-architect, code-analyst, performance-engineer, security-expert, devops-engineer, test-engineer, ui-ux-designer | *(omitted — deny by default)* | *(omitted — deny by default)* | No |
| Implementer | developer-prime, developer-fast | `allow` | `allow` | Yes |

#### Bash Permission Model (defined in `opencode.json`)

All agents share a common baseline of `allow` rules defined in `opencode.json`, with role-specific additions:

**Universal (all agents — always `allow`)**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [the-perfect-developer/the-perfect-opencode](https://github.com/the-perfect-developer/the-perfect-opencode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-26 -->
