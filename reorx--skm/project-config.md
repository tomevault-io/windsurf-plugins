---
trigger: always_on
description: A CLI tool that manages AI agent skills by cloning GitHub repos, detecting skills via `SKILL.md`, and symlinking them to agent directories based on a central YAML config.
---

# SKM - Skill Manager

A CLI tool that manages AI agent skills by cloning GitHub repos, detecting skills via `SKILL.md`, and symlinking them to agent directories based on a central YAML config.

## Tech Stack

- Python 3.12+, uv, click, pyyaml, pydantic
- Git operations via subprocess (unified `run_cmd` helper raises `click.ClickException` on failure)
- Tests: pytest

## Project Structure

```
src/skm/
├── cli.py              # Click CLI entry point (group + subcommands)
├── types.py            # Pydantic data models + constants
├── config.py           # Load skills.yaml → SkmConfig
├── lock.py             # Read/write skills-lock.yaml
├── detect.py           # Walk cloned repos for SKILL.md files
├── git.py              # Clone, pull, fetch, commit SHA helpers (unified run_cmd error handling)
├── utils.py            # Utility functions (compact_path)
├── linker.py           # Symlink skills to agent dirs, resolve includes/excludes
└── commands/
    ├── install.py      # Clone repos, detect skills, link to agents, update lock
    ├── list_cmd.py     # Print installed skills from lock file
    ├── check_updates.py # Fetch remotes, compare commits, show available updates
    └── update.py       # Pull latest for a skill's repo, re-link, update lock
tests/
├── test_types.py        # Pydantic model validation
├── test_config.py       # Config loading, error handling
├── test_lock.py         # Lock file I/O
├── test_detect.py       # Skill detection logic (local + network tests against real repos)
├── test_git.py          # Git operations (clone, commit retrieval, clone failure handling)
├── test_linker.py       # Symlink creation, agent filtering
├── test_install.py      # Install command unit tests
└── test_cli_e2e.py      # End-to-end CLI tests for all commands
```

## Key Paths

- **Config:** `~/.config/skm/skills.yaml` — YAML dict with `packages` (repo list) and optional `agents.default`
- **Lock:** `~/.config/skm/skills-lock.yaml` — tracks installed skills, commits, symlink paths
- **Store:** `~/.local/share/skm/skills/` — cloned repos cached here
- **Agent dirs:** Skills are symlinked into each agent's skill directory (e.g. `~/.claude/skills/`, `~/.codex/skills/`)

## Architecture

Config-driven: parse `skills.yaml` → clone repos to store → detect skills by walking for `SKILL.md` → symlink to agent dirs → write lock file.

Each command function (`run_install`, `run_list`, etc.) accepts explicit paths and agent dicts as parameters, making them testable with `tmp_path` fixtures without touching real filesystem locations.

## Error Handling

All git subprocess calls go through `run_cmd()` in `git.py`, which captures stdout/stderr and raises `click.ClickException` on non-zero exit codes. This produces user-friendly error messages instead of raw `CalledProcessError` tracebacks.

## Path Handling

Paths stored in `skills-lock.yaml` (e.g. `linked_to`) use `compact_path()` from `utils.py` to replace the home directory with `~`, avoiding exposure of usernames. When reading these paths back for filesystem operations, `Path.expanduser()` is used.

## CLI Commands

- `skm install` — Clone repos (idempotent: skips pull if already cloned), detect skills, create symlinks, remove stale links, update lock. Treats `skills.yaml` as declarative state: removes links for skills dropped from config or agents changed by includes/excludes. Only removes links tracked in the lock file — manually created files in agent dirs are never touched.
- `skm list` — Show installed skills and their linked paths from lock file
- `skm check-updates` — Fetch remotes, compare against locked commits, show changelog
- `skm update <skill_name>` — Pull latest for a skill's repo, re-link, update lock

## Config Format (skills.yaml)

Top-level YAML dict with `packages` and optional `agents`:

```yaml
agents:
  default:                   # optional: select which KNOWN_AGENTS are active (omit = all)
    - claude
    - standard

packages:
  - repo: https://github.com/vercel-labs/agent-skills
    skills:                  # optional: filter to specific skills (omit = all)
      - react-best-practices
    agents:                  # optional: further filter agents for this package
      excludes:
        - standard
  - repo: https://github.com/blader/humanizer   # installs all detected skills to default agents
```

`agents.default` selects which agents from `KNOWN_AGENTS` are used as the base set. Per-package `agents.includes/excludes` then filters from that base set.

## Skill Detection

A skill is a directory containing a `SKILL.md` file with YAML frontmatter including a `name` field. Detection order:
1. Root `SKILL.md` → singleton skill (the repo itself is the skill)
2. `./skills/` subdirectory exists → walk its children
3. Otherwise → walk all subdirectories from repo root
4. Stop descending once `SKILL.md` is found (no nested skill-in-skill)

## Known Agents

Defined in `src/skm/types.py` as `KNOWN_AGENTS`:
- `standard` → `~/.agents/skills`
- `claude` → `~/.claude/skills`
- `codex` → `~/.codex/skills`
- `openclaw` → `~/.openclaw/skills`

## Testing

### Running Tests

```bash
uv sync
uv run pytest -v              # all tests

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [reorx/skm](https://github.com/reorx/skm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
