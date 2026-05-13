---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

Plan-Build-Run is a **Claude Code plugin** that provides a structured development workflow. It solves context rot — quality degradation as Claude's context window fills up — through disciplined subagent delegation, file-based state, and goal-backward verification. Users invoke `/pbr:*` slash commands (skills) that orchestrate specialized agents via `Task()`.

## Critical Rules

- **NEVER add AI co-author lines** to git commits or PRs. No `Co-Authored-By: Claude` or similar. Only add co-author lines referencing actual human contributors.
- **NEVER inline agent definitions** into skill prompts. Use `subagent_type: "pbr:{name}"` — Claude Code auto-loads agent definitions from `agents/`. Reading agent `.md` files wastes main context.
- **Always use PBR workflow commands** (`/pbr:*` skills) for development work, not ad-hoc manual fixes. PBR skills enforce state tracking, verification, and artifact creation that manual edits skip. When hooks don't fire (e.g., direct file edits outside Claude Code), workflow integrity degrades silently.
- **Post-work checklist** — before finishing any task or session, verify: (1) `npm test` passes, (2) no uncommitted changes (`git status` is clean), (3) file paths in artifacts are correct and files exist. This catches drift that accumulates during long sessions.

## Commands

```bash
npm test                    # Run all Jest tests
npm run test:coverage       # Jest with coverage enforcement
npm run test:dashboard      # Vitest dashboard tests
npm run lint                # ESLint on hooks and tests
npm run build:hooks         # Bundle hooks for distribution
npm run sync:generate       # Generate derivative plugins
npm run sync:verify         # Verify derivative consistency
npm run dashboard           # Launch dashboard UI
npm run dashboard:install   # Install dashboard dependencies
```

Coverage thresholds (enforced in `jest.config.cjs`): 55% statements, 47% branches, 58% functions, 56% lines.

Dashboard (separate dependency tree):
```bash
npm run dashboard:install                   # One-time install of dashboard deps
npm run dashboard -- --dir /path/to/project # Launch dashboard for a project
```

Load the plugin locally for manual testing:
```bash
claude --plugin-dir .
```

CI runs on Node 20/22 across Windows, macOS, and Linux. All three platforms must pass.

## Architecture

Three layers:

### Skills (`plan-build-run/skills/{name}/SKILL.md`)

Markdown files with YAML frontmatter defining slash commands (`/pbr:new-project`, `/pbr:plan-phase`, etc.). Each SKILL.md is a complete prompt that tells the orchestrator what to do. Skills read state, interact with the user, and spawn agents.

46 skills: audit, begin, build, config, continue, dashboard, debug, discuss, do, explore, health, help, import, intel, milestone, note, pause, plan, profile, quick, resume, review, scan, setup, status, statusline, test, todo, undo, and more.

### Agents (`agents/{name}.md`)

Markdown files with YAML frontmatter defining specialized subagent prompts. Agents run in fresh `Task()` contexts with clean 200k token windows. Spawned via `subagent_type: "pbr:{name}"` — auto-loaded by Claude Code.

18 agents: audit, codebase-mapper, debugger, dev-sync, executor, general, integration-checker, intel-updater, nyquist-auditor, plan-checker, planner, researcher, roadmapper, synthesizer, ui-checker, ui-researcher, verifier.

### Hook Scripts (`plugins/pbr/scripts/*.js`)

66 Node.js hook scripts that fire on Claude Code lifecycle events. Configured in `plugins/pbr/hooks/hooks.json`. All use CommonJS, must be cross-platform (`path.join()`, not hardcoded separators), and log via `logHook()` from `hook-logger.js`.

**Dispatch pattern**: Several hooks use dispatch scripts that fan out to sub-scripts based on the file being written/read:

| Hook Event | Entry Script | Delegates To |
|------------|-------------|-------------|
| SessionStart | progress-tracker.js | — (injects project state) |
| PostToolUse (Write\|Edit) | post-write-dispatch.js | check-plan-format.js, check-roadmap-sync.js, check-state-sync.js |
| PostToolUse (Write\|Edit) | context-bridge.js | — (context tier tracking) |
| PostToolUse (Write\|Edit) | graph-update.js | — (architecture graph) |
| PostToolUse (Write\|Edit) | architecture-guard.js | — (dependency violations) |
| PostToolUse (Write\|Edit) | suggest-compact.js | — (context budget warnings) |
| PostToolUse (Read\|Glob\|Grep) | track-context-budget.js | — (tracks reads for budget) |
| PostToolUse (Bash) | post-bash-triage.js | — (test output triage) |
| PostToolUse (Bash) | context-bridge.js | — (context tier tracking) |
| PostToolUse (Task) | check-subagent-output.js | — (validates agent output) |
| PostToolUse (Task) | context-bridge.js | — (context tier tracking) |
| PostToolUse (AskUserQuestion) | track-user-gates.js | — (gate tracking) |
| PostToolUseFailure | log-tool-failure.js | — (logs failures) |
| PreToolUse (Bash) | pre-bash-dispatch.js | validate-commit.js, check-dangerous-commands.js |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SienkLogic/plan-build-run](https://github.com/SienkLogic/plan-build-run) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
