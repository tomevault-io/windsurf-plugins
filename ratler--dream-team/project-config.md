---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

Dream Team is a Claude Code **marketplace plugin** that provides structured planning and execution for development projects. It follows the brainstorm → spec → build workflow across three execution tiers: sequential, delegated (sub-agents), and team (parallel Claude instances).

Installed as: `dream-team@dream-team-marketplace` (local marketplace via `.claude-plugin/marketplace.json`).

## Commands

```bash
# Run all tests
make test

# Run a single test
node tests/test_session_start.js

# Install as plugin (local marketplace)
claude plugin marketplace add Ratler/dream-team-marketplace
claude plugin install dream-team@dream-team-marketplace
```

Tests are plain Node.js scripts (no test framework) that use `execSync` to invoke hooks and assert on JSON output. Hook tests that expect `exit(1)` need try/catch with `err.stdout` parsing.

## Architecture

This is a Claude Code plugin, not a Node.js application. There is no `package.json` or build step. All files live at repo root (required by marketplace layout).

### Plugin Structure

- **`commands/*.md`** — Slash-command autocomplete entries. Thin wrappers that read and delegate to the matching skill via `${CLAUDE_PLUGIN_ROOT}/skills/<name>/SKILL.md`. Do NOT set `disable-model-invocation: true` on commands that delegate to skills (breaks the Skill tool chain).
- **`skills/*/SKILL.md`** — Actual skill logic. YAML frontmatter defines name, description, argument hints, and Stop hooks. The `allowed-tools` frontmatter field controls tool access (`disallowed-tools` is not a valid field).
- **`agents/*.md`** — Agent definitions with model, color, and behavioral instructions. Models: builder=opus, researcher=sonnet, architect=opus, reviewer=sonnet, tester=sonnet, validator=haiku, debugger=opus, security-reviewer=opus, docs=sonnet, scout=haiku, merger=sonnet. Builder and debugger agents use `isolation: "worktree"` for git worktree isolation in delegated/team modes. Builder, reviewer, architect, and docs agents use `memory: project` for persistent cross-session knowledge. Read-only agents (researcher, architect, reviewer, security-reviewer, validator, scout) use `disallowedTools: Write, Edit, NotebookEdit` to enforce read-only access. All agents include `[agent-type: X]` in their report format and write reports to the task description via `TaskUpdate(description: ...)`.
- **`hooks/*.js`** — JavaScript hooks (Node.js, no external deps). `hooks.json` registers SessionStart, PreToolUse, TaskCompleted, and PostCompact hooks. Stop hooks are declared in skill frontmatter under the `hooks:` key.
- **`lib/build-state.js`** — State persistence library. Functions for creating, reading, and updating per-task state files in `specs/.build-state/<spec-name>/`. Used by hooks and the progress dashboard.
- **`bin/progress.js`** — CLI progress dashboard. Run `node bin/progress.js <spec-path>` from any terminal to see build progress without needing a Claude session.
- **`templates/spec-template.md`** — Shared spec template with conditional sections per execution mode and YAML frontmatter.
- **`specs/`** — Generated spec files (date-prefixed: `YYYY-MM-DD-<name>.md`). Build state is persisted under `specs/.build-state/<spec-name>/` with one JSON file per task plus `_meta.json` for build metadata.
- **`.claude-plugin/plugin.json`** — Plugin manifest. Omit empty string fields (e.g., `homepage: ""` fails URL validation).
- **`.claude-plugin/marketplace.json`** — Local marketplace definition.

### Hook Protocol

Hooks are plain Node.js scripts that read stdin and write JSON to stdout:
- **SessionStart** (`session_start.js`): Async hook. Outputs `{hookSpecificOutput: {hookEventName, additionalContext}}`. Injects usage guide into session context.
- **Stop hooks** (`validate_spec_exists.js`, `validate_build_complete.js`, `validate_spec_sections.js`): Exit 0 + `{"result":"continue"}` to allow, exit 1 + `{"result":"block","reason":"..."}` to block. Stop hooks in skill frontmatter appear to be working as of Claude Code 2.1.49 (previously broken upstream — claude-code#19225). All Stop hooks log `last_assistant_message` to stderr when available (visible in `claude --debug`).
- **TaskCompleted** (`validate_task_completed.js`): Always exits 0. Three responsibilities: (1) appends audit log entry to `~/.claude/dream-team/logs/<sanitized-cwd>.jsonl`, (2) persists completed task state to `specs/.build-state/<spec-name>/<task-id>.json` when an active build exists, (3) runs `markdown-table-formatter` on any `.md` files listed in the task description (noop if formatter not installed). Uses `DREAM_TEAM_LOG_DIR` env var for test override.
- **PostCompact** (`postcompact_checkpoint.js`): Always exits 0. Increments the `compactions` counter in `specs/.build-state/<spec-name>/_meta.json` when an active build exists. Tracks context compaction frequency as a signal for long-running or stalled builds.

### Skill Pipeline

1. `/dream-team:plan` — Interactive brainstorming, produces no files
2. `/dream-team:spec-{sequential,delegated,team}` — Writes spec to `specs/` from conversation context

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Ratler/dream-team](https://github.com/Ratler/dream-team) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-09 -->
