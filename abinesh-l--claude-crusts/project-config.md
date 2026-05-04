---
trigger: always_on
description: You are working on **claude-crusts** (the npm package name and CLI binary). The framework is called **CRUSTS** (always uppercase). It analyzes Claude Code's context window by classifying every token into 6 categories: Conversation, Retrieved, User, System, Tools, State. It tells users WHY their context is filling up and WHAT to do about it. Fully offline, zero API calls.
---

# CLAUDE.md — claude-crusts: Context Window Analyzer for Claude Code

## Identity

You are working on **claude-crusts** (the npm package name and CLI binary). The framework is called **CRUSTS** (always uppercase). It analyzes Claude Code's context window by classifying every token into 6 categories: Conversation, Retrieved, User, System, Tools, State. It tells users WHY their context is filling up and WHAT to do about it. Fully offline, zero API calls.

## CLI Commands

Binary name: `claude-crusts`. Entrypoint: `src/index.ts`.

**Slash command**: `/crusts` inside Claude Code (via `.claude/commands/crusts.md`) — runs analyze and gives actionable advice.

```
claude-crusts analyze [session-id]              — 6-category breakdown + waste detection + recommendations
claude-crusts waste [session-id]                — Detailed waste report with per-file analysis + top 5 consumers
claude-crusts fix [session-id]                  — 3 pasteable prompt blocks (session, CLAUDE.md, /compact)
claude-crusts timeline [session-id]             — Message-by-message context growth with compaction markers
claude-crusts list                              — All discovered sessions (age, size, project)
claude-crusts compare <session-a> <session-b>   — Side-by-side comparison with per-category deltas + insights
claude-crusts lost [session-id]                 — What was lost during compaction (files, conversations, tools, instructions)
claude-crusts watch [session-id]                — Live-monitor a session with compact dashboard
claude-crusts report [session-id]               — Generate standalone report (HTML or Markdown)
claude-crusts calibrate                         — Cross-reference against /context output
claude-crusts trend                              — Cross-session trends (sparkline, averages, direction)
claude-crusts tui [session-id]                   — Interactive REPL shell with tab completion + clipboard copy
claude-crusts status [session-id]                — One-line context health (fast path, used by hooks)
claude-crusts hooks enable|disable|status        — Manage Claude Code hook integration
```

Global flags: `--json`, `--project <name>`, `--path <path>`, `--verbose`
Subcommand flags: `--until <n>` on analyze/waste/timeline
Report flags: `--format <html|md>` (default: html), `--compare <id>` for comparison report, `--output <path>` for custom file path
Watch flags: `--interval <ms>` polling interval (default: 2000)
Trend flags: `--limit <n>` number of sessions (default: 50)

## Rules

- TypeScript strict mode, Bun runtime
- Commander.js for CLI, chalk for colors, cli-table3 for tables
- NO React/Ink — lightweight terminal output only
- Stream-parse JSONL files (they can be multi-MB)
- Every function must have JSDoc comments
- MIT license

## Format

- 2-space indentation, single quotes, semicolons
- kebab-case file naming
- Named exports only (no default exports)
- Explicit `.ts` import extensions (Bun bundler resolution)
- `import type` for type-only imports (`verbatimModuleSyntax: true`)
- Array indexing requires `!` postfix (`noUncheckedIndexedAccess: true`)

## Architecture

```
scanner.ts → classifier.ts → waste-detector.ts → recommender.ts → renderer.ts
                  ↑                                                     ↑
              analyzer.ts (orchestrates)                         calibrator.ts
                  ↓                                             comparator.ts
              trend.ts (history)                                lost-detector.ts
                                                                watcher.ts
                                                                tui.ts
                                                                clipboard.ts
                                                                hooks.ts
                                                                html-report.ts
                                                                md-report.ts
```

Supporting: `types.ts`, `built-in-tools.ts`

### File Responsibilities

- **types.ts**: All shared types and interfaces (including ComparisonResult, CategoryDelta, TrendRecord, TrendSummary, SkillInfo)
- **index.ts**: CLI entrypoint with 14 Commander.js commands (including hooks subcommands)
- **analyzer.ts**: Pipeline orchestrator, project path decoding, trend recording
- **scanner.ts**: Session discovery, JSONL streaming, config readers (MCP, memory, skills)
- **classifier.ts**: Core engine — classification, token estimation, compaction detection, derived overhead, auto-trim
- **waste-detector.ts**: 6 waste detection rules (edit-aware)
- **recommender.ts**: 7 recommendation patterns + fix prompt generator
- **renderer.ts**: 8 render functions (dashboard, timeline, list, waste, fix, comparison, lost, trend). Bar chart guarantees ≥1 filled block for categories ≥1%
- **calibrator.ts**: /context parser, calibration storage, comparison. Exports `CRUSTS_DIR` (~/.claude-crusts)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Abinesh-L/claude-crusts](https://github.com/Abinesh-L/claude-crusts) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
