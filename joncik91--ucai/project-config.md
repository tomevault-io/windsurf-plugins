---
trigger: always_on
description: A Claude Code plugin that leverages native architecture — commands, agents, hooks,
---

# Ucai — Use Claude Code As Is

## Overview
A Claude Code plugin that leverages native architecture — commands, agents, hooks,
and skills — exactly as Anthropic designed them. v2.2 adds the never-forget
enforcement engine — programmatic phase enforcement via ContingencyEngine with
dependencies, logic gates, shadow tasks, and audit trail. v2.0 added autonomous
execution — `/ship` (zero-gate spec-to-PR pipeline), `/bootstrap` (infrastructure
scaffolding), PostToolUse auto-formatting, deterministic test execution, and
lessons consolidation. Built on the Cherny methodology: persistent task tracking,
self-improvement loop, TDD integration, and elegance checkpoints.

## Tech Stack
- **Runtime**: Node.js 18+ (CommonJS, zero external dependencies)
- **Plugin format**: Markdown (YAML frontmatter) + JSON configs
- **Platform**: Windows/Linux/macOS, CI matrix: Node 18 × 20 on ubuntu + windows

## Development Commands
No build step. Validation only:
```bash
# Local dev
claude --plugin-dir ./ucai

# JSON syntax
node -e "JSON.parse(require('fs').readFileSync('.claude-plugin/plugin.json', 'utf8'))"
node -e "JSON.parse(require('fs').readFileSync('hooks/hooks.json', 'utf8'))"
node -e "JSON.parse(require('fs').readFileSync('.claude-plugin/marketplace.json', 'utf8'))"

# JS syntax (all handlers + scripts)
for file in hooks/handlers/*.js scripts/*.js; do node -c "$file" || exit 1; done

# Smoke test
output=$(node hooks/handlers/sessionstart-handler.js)
node -e "const o=JSON.parse(process.argv[1]); if(!o.hookSpecificOutput) process.exit(1)" "$output"
```

## Architecture

### Layers
- **commands/** — Orchestration: phased workflows with user approval gates. Write/Edit only here.
- **agents/** — Execution: read-only analysis workers, spawned in parallel via Task tool.
- **hooks/handlers/** — Lifecycle: context injection, state management, config protection.
- **skills/** — Knowledge: progressive disclosure domain expertise, loaded on-demand.
- **scripts/** — Utilities: `setup-iterate.js`, `setup-ship.js`, `detect-infra.js`, `run-tests.js`, `consolidate-lessons.js`.
- **scripts/lib/never-forget/** — Vendored never-forget engine (ESM, zero deps): dependency validation, logic gates, shadow tasks, audit trail.

### Commands (12 slash commands)
`/init`, `/plan`, `/build` (8-phase, guided), `/ship` (9-phase, autonomous), `/bootstrap`,
`/debug`, `/review`, `/docs`, `/release`, `/iterate`, `/cancel-iterate`, `/cancel-ship`

### Agents (8, all read-only — no Write/Edit)
| Agent | Model | Max Turns | Purpose |
|-------|-------|-----------|---------|
| `project-scanner` | haiku | — | Fast structure + convention analysis |
| `explorer-haiku` | haiku | 12 | Quick scan (~8 tool calls) |
| `explorer` | sonnet | 20 | Balanced analysis (~15 tool calls) |
| `explorer-opus` | opus | 30 | Deep analysis (~25 tool calls) |
| `architect` | opus | — | Feature architecture + implementation blueprint |
| `reviewer` | sonnet | — | Code review with confidence scoring |
| `reviewer-opus` | opus | — | Deep review for subtle/high-impact issues |
| `verifier` | sonnet | — | Acceptance criteria validation |

### Hooks (8 lifecycle handlers, all in `hooks/handlers/`)
| Hook | Handler | Purpose |
|------|---------|---------|
| SessionStart | `sessionstart-handler.js` | Git branch, iterate/ship status, task progress, lessons, specs, skills |
| PostToolUse (Write\|Edit) | `posttooluse-format-handler.js` | Auto-format files after write/edit operations |
| PreToolUse (Write\|Edit) | `pretooluse-guard.js` | Guard config files (ask before modifying) |
| UserPromptSubmit | `userpromptsubmit-handler.js` | Inject iterate/ship context + active task from todo.md |
| Stop | `stop-handler.js` | Block exit to continue iterate loop or ship pipeline |
| SubagentStop | `subagent-stop-handler.js` | Block on empty output; inject 1-line preview |
| PreCompact | `precompact-handler.js` | Surface iterate/ship state, task progress, latest lesson before compaction |
| SessionEnd | `session-end-handler.js` | Delete stale iterate/ship state + formatter cache on termination |

### Iterate Loop
State file: `.claude/ucai-iterate.local.md` (gitignored). YAML frontmatter holds
`iteration`, `max_iterations`, `completion_promise`; body holds the task.
Stop hook reads state → feeds task back → checks limits → continues or exits.

### Ship Pipeline
State file: `.claude/ucai-ship.local.md` (gitignored). YAML frontmatter holds
`phase`, `milestone`, `fix_attempts`, `max_fix_attempts`, `test_cmd`, `lint_cmd`,
`format_cmd`, `spec_source`, `worktree`, `no_pr`, `ci_watch`; body holds the spec.
Stop hook reads state → feeds phase-aware continuation → resumes pipeline.
Priority: iterate > ship > normal exit.

### Context Chain
- `/plan` → `.claude/project.md` + `.claude/requirements.md` (with build order)
- `/plan <feature>` → `.claude/frds/<slug>.md` (never overwritten)
- `/build`, `/debug` → `tasks/todo.md` (overwritten per session), `tasks/lessons.md` (append-only)
- `/ship` → `.claude/ucai-ship.local.md` (pipeline state), uses same context chain as `/build`
- All commands auto-load whatever spec files exist in `.claude/`
- `/build`, `/debug`, `/review`, `/docs`, `/ship` load `tasks/lessons.md` for known patterns

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Joncik91/ucai](https://github.com/Joncik91/ucai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
