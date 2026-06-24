---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
bun test src/        # Run tests from source (no build step)
bun run typecheck    # Type-check without emitting
```

Test files live alongside source with `.test.ts` suffixes (e.g. `src/commands/can-exit.test.ts`). Bun's test runner discovers and runs them directly from TypeScript source.

The `kata` shell script at the repo root is the CLI entry point. It runs `bun src/index.ts` directly — no build step required.

## Architecture

**kata-wm** is a TypeScript CLI published as an npm package (`@codevibesmatter/kata`). It wraps Claude Code projects with structured session modes, phase task enforcement, and stop hooks.

### Source layout (`src/`)

| Directory | Purpose |
|-----------|---------|
| `index.ts` | CLI dispatcher — maps `kata <command>` to handler functions; also re-exports the programmatic API |
| `commands/` | One file per CLI command (`enter.ts`, `exit.ts`, `hook.ts`, `setup.ts`, etc.) |
| `commands/enter/` | Sub-modules for the `enter` command: `task-factory.ts` (native task creation), `guidance.ts`, `template.ts`, `spec.ts` |
| `session/lookup.ts` | Project root discovery, session ID resolution, template path resolution |
| `state/` | Zod schema (`schema.ts`), reader/writer for `SessionState` JSON |
| `config/` | `kata-config.ts` loads `.kata/kata.yaml` |
| `validation/` | Phase/template validation |
| `yaml/` | YAML frontmatter parser for template files |
| `utils/` | Workflow ID generation, session cleanup, timestamps |
| `testing/` | Test utilities — mock sessions, hook runners, assertions, pre-built scenarios |

### Runtime data layout

All kata-owned config lives under `.kata/`. Claude-owned files (`.claude/settings.json`, `.claude/skills/`) remain in `.claude/`.

| Path | Contents |
|---|---|
| `.kata/kata.yaml` | Project config (modes with rules, settings) |
| `.kata/ceremony.md` | Shared workflow instructions (commit, PR, branch, env-check, tests) |
| `.kata/sessions/{sessionId}/state.json` | Per-session `SessionState` |
| `.kata/templates/` | Project-level template overrides (optional — batteries fallback if absent) |
| `.kata/prompts/` | Review prompt templates (customizable) |
| `.kata/verification-evidence/` | Verify-phase output |
| `~/.claude/tasks/{sessionId}/` | Native task files (Claude-owned) |
| `.claude/settings.json` | Hook registration (Claude-owned) |
| `~/.claude/skills/kata-{name}/` | User-scoped methodology skills (kata-code-impl, kata-code-review, etc.) |
| `.claude/skills/` | Project-level skill overrides (optional — takes precedence over user-scoped) |
| `planning/spec-templates/` | Spec document stubs |

### Hook architecture

Hooks are registered in `.claude/settings.json` and call `kata hook <name>`. Each hook reads Claude Code's stdin JSON, extracts `session_id`, and outputs a JSON decision. The session ID from hook stdin **must** be forwarded as `--session=ID` to any subcommand — there is no automatic session detection at runtime.

| Hook event | Command | Role |
|------------|---------|------|
| `SessionStart` | `kata hook session-start` | Init session registry, inject mode context + rules |
| `UserPromptSubmit` | `kata hook user-prompt` | Detect mode intent, suggest entering a mode |
| `PreToolUse` | `kata hook pre-tool-use` | Consolidated: mode-gate, session ID, gate eval, task-deps, task-evidence |
| `Stop` | `kata hook stop-conditions` | Block exit while conditions are unmet, detect active agents |

### Mode and template system

Mode definitions live in `kata.yaml` under the `modes:` key. Each mode references a template filename with YAML frontmatter defining phases (with stages, skills, gates, and expansion types).

**Template resolution (dual resolution):**
1. Project-level: `.kata/templates/{name}.md` (optional overrides)
2. Package-level: `batteries/templates/{name}.md` (fallback)

Templates resolve at runtime — no project copies needed. Projects only create `.kata/templates/` files to override specific templates.

**Skill resolution:**
- User-scoped: `~/.claude/skills/kata-{name}/` (installed by `kata setup`/`kata update`)
- Project-scoped: `.claude/skills/kata-{name}/` (optional overrides, takes precedence)

**Ceremony:** `.kata/ceremony.md` contains shared workflow instructions (commit patterns, PR creation, branch naming, env checks, tests). Created by `kata setup`, updated by `kata update`.

**Other batteries content** (prompts, interviews, spec-templates, etc.) is still copied to the project by `kata setup`.

### Key dependencies

- **zod** — schema validation for `SessionState`, `ModeConfig`, and config files
- **js-yaml** — YAML parsing for `modes.yaml`, `wm.yaml`, and template frontmatter

## Data-driven design principles

**No hardcoded mode names in logic.** Mode behavior is driven by fields in `kata.yaml` mode definitions:
- `issue_handling: "required" | "none"` — whether mode entry requires a GitHub issue
- `stop_conditions: string[]` — which exit checks to run (`tasks_complete`, `committed`, `pushed`, `tests_pass`, `feature_tests_added`, `doc_created`, `spec_valid`). Empty array = can always exit.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ivebenfreed/kata](https://github.com/ivebenfreed/kata) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-24 -->
