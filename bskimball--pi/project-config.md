---
trigger: always_on
description: This directory (`~/.pi`) is the workspace for the user's custom Pi configuration. Requests here concern Pi itself — its agents, prompts, skills, extensions, models, MCP integrations, and themes — unless the user explicitly names another repository.
---

# AGENTS.md — Custom Pi Configuration

## Purpose

This directory (`~/.pi`) is the workspace for the user's custom Pi configuration. Requests here concern Pi itself — its agents, prompts, skills, extensions, models, MCP integrations, and themes — unless the user explicitly names another repository.

Do not redirect requests to an unrelated application repository based on terms such as "the app," "the UI," or references to earlier work. If the user explicitly names another project, locate it, read its repository instructions, and inspect its worktree before making changes.

## Layout

```text
agent/
├── SYSTEM.md              # base system prompt
├── settings.json          # model, theme (currently claude-dark), tuiMode, compaction
├── models.json / mcp.json # provider + native Pi MCP configuration
├── agents/                # specialist briefs (_shared*.md are composed in)
├── prompts/               # slash-command prompts (inactive/ = extension-owned; poteto/ = /poteto playbooks, not slash commands)
├── skills/                # local skills
├── themes/                # apex-dark.json, claude-dark.json
├── harness/                # runtime state: global memory, model circuits
├── extensions/             # see below
└── logs/                  # render / crash / lifecycle / lsp traces
reference/                 # source material, not runtime data
```

## Extension Architecture

Pi discovers extensions two ways: a bare `*.ts` file in `agent/extensions/`, or a directory whose `package.json` declares `pi.extensions`. Everything under a directory that is not a declared entry point is private support code.

```text
agent/extensions/
├── apex/            → apex-ui.ts          Apex UI (shark Observatory, braille indicator, sonar footer)
├── claude/          → claude-ui.ts        Claude UI (star motifs, Claude verbs, Claude Code footer)
├── hal/             → hal-ui.ts           HAL UI (orb landing, square glyphs, HAL panel footer)
├── task/            → amp-task.ts, async-task.ts   sync `task` + async task_* RPC workers
├── lsp/             → index.ts            language-server navigation
├── bg-process.ts    + bg-process/         bg_start/status/list/kill
├── powershell.ts    + powershell/         direct PowerShell child process
├── crash-logger.ts  + crash-logger/       crash/lifecycle logs, terminal restore, segmenter shield
├── continual-memory.ts + continual-memory/  memory_list / memory_write
├── prompt-commands.ts + prompt-commands/  /browser, /deploy, /orchestrate
├── worktree.ts   + worktree/               isolated Git worktree add/list/remove
├── read-guard.ts                          duplicate-image + downscale guard
├── user-profile.ts                        private user context injection
├── web-search.ts                          Exa search + fetch_content
├── jev/                     → index.ts              Advisory Jev Choice/Score/Noul classifier (single tool)
├── at-path-complete.ts                    scoped @ listing for gitignored paths
├── claude-bridge-sonnet-5-5.ts            TEMPORARY: adds claude-sonnet-5-5 to claude-bridge; delete once pi-ai ships it (it notifies)
└── test/                                  cross-extension tests
```

### Each Extension Stands On Its Own

This is the load-bearing invariant, enforced by `extensions/test/extension-discovery.test.ts`:

- **No cross-extension source imports.** Every relative import in an entry point's closure must stay under that extension's own directory. Shared UI presentation lives in `packages/ui-kit` (`@pi/ui-kit`), not `agent/extensions/shared` — the test still asserts `extensions/shared` does not exist. Headless helpers (`last-phase.ts`, `terminal-restore.ts`, `process-tree-kill.ts`, `agent-discovery.ts`, `segmenter-safety.ts`) stay duplicated per owner.
- **One entry point per extension, discovered once.** Support code lives in `internal/`, `runtime/`, `presentation/`, `observatory/`, or `test/` so it is never loaded as a second extension.
- **Deleting an extension directory + its entry file removes the feature cleanly**, with no dangling imports elsewhere.

When editing a duplicated helper, decide deliberately whether the change belongs to one owner or all of them, and apply it per owner.

### Three installable UIs

`apex/`, `claude/`, and `hal/` are separately discovered UI extensions. Shared receipts, layout, todo tools, and the single `ToolExecutionComponent` wrap live in `packages/ui-kit`. Deleting one UI directory uninstalls that look. See `CONTEXT.md` for presentation ownership.

- `PI_UI_CHROME=0` is the installation-wide presentation opt-out (`PI_APEX_UI=0` remains a deprecated alias; `PI_UI_CHROME` wins when both are set): it disables custom styling, chrome, and render hooks. Kit-owned tools remain registered and executable. The todo panel stays mounted as a plain, uncolored list.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bskimball/pi](https://github.com/bskimball/pi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
