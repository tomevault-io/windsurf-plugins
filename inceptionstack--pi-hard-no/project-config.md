---
trigger: always_on
description: A [pi](https://github.com/badlogic/pi-mono) extension that automatically reviews code changes after each agent turn. It spawns a separate, isolated pi reviewer instance that reads changed files, examines diffs, and feeds findings back to the main agent for fixes. Includes an "architect" review that checks cross-file consistency after multi-file changes pass individual reviews.
---

# AGENTS.md — Agent Guide for pi-hard-no

## What is this project?

A [pi](https://github.com/badlogic/pi-mono) extension that automatically reviews code changes after each agent turn. It spawns a separate, isolated pi reviewer instance that reads changed files, examines diffs, and feeds findings back to the main agent for fixes. Includes an "architect" review that checks cross-file consistency after multi-file changes pass individual reviews.

## Quick orientation

```
pi-hard-no/
├── index.ts              ← Extension entry point, pi wiring, UI (~760 lines)
├── orchestrator.ts       ← Auto-review state machine & sequencing (~420 lines)
├── commands.ts           ← Manual review commands (/review N, /review-all, etc.)
├── reviewer.ts           ← Spawns pi session, runs review, parses verdict
├── judge.ts              ← Opt-in bash-command classifier (duplicate-review suppressor)
├── message-sender.ts     ← sendReviewResult — formats & sends review messages
├── context.ts            ← Builds review content (4 fallback paths)
├── changes.ts            ← Change detection, tool call classification
├── prompt.ts             ← Review prompt construction (3-part structure)
├── architect.ts          ← Architect prompt + shouldRunArchitectReview
├── settings.ts           ← Config loading from .hardno/ dirs
├── ignore.ts             ← Gitignore-style pattern matching
├── git-roots.ts          ← Multi-repo git root detection
├── helpers.ts            ← Pure utility functions
├── logger.ts             ← File logger + structured JSON review records
├── review-display.ts     ← TUI widget (ASCII art + file progress)
├── scaffold.ts           ← Template content for /scaffold-review-files
├── default-review-rules.md ← Default review criteria (OWASP, SOLID, DRY, etc.)
├── test/                 ← 352 tests across 13 files (vitest)
└── .hardno/       ← Local config (settings.json, review-rules.md, etc.)
```

## Key commands

```bash
npm run check          # Full CI: typecheck + lint + format + tests
npm run test           # Run tests (vitest)
npm run test:watch     # Watch mode
npm run typecheck      # TypeScript type checking (tsc --noEmit)
npm run lint           # ESLint
npm run lint:fix       # ESLint with auto-fix
npm run format         # Prettier format
npm run format:check   # Prettier check
```

## Tech stack

- **Language:** TypeScript (ES2022, ESNext modules)
- **Runtime:** Node.js (runs inside pi agent)
- **Testing:** Vitest
- **Linting:** ESLint 9 + typescript-eslint
- **Formatting:** Prettier
- **Dependency:** `@mariozechner/pi-coding-agent` (peer dep — the pi SDK)
- **No runtime dependencies** — only peer dep on the pi SDK
- **Key design pattern:** Orchestrator returns outcomes, index.ts renders them (clean separation of logic from UI)

## Git workflow

- **Never amend commits. Always append new commits.**
- Run `npm run check` before committing to catch type errors, lint issues, and test failures.
- Each commit should be self-contained and pushable.

## Architecture at a glance

The extension hooks into pi's lifecycle events:

```
session_start → load config from .hardno/ dirs
tool_execution_start/end → track which files the agent modifies
agent_end → if files changed, trigger review pipeline
```

The review pipeline:

```
1. Detect changed files (modifiedFiles + tool call paths + git roots)
2. Build review content (4 fallback paths in context.ts)
3. Spawn isolated pi reviewer session (reviewer.ts)
4. Parse verdict (LGTM or ISSUES_FOUND)
5. If issues → feed back to main agent → agent fixes → re-review (loop)
6. If LGTM → optionally run architect review (for multi-file changes)
```

## How the modules connect

```
index.ts (pi wiring, UI, renderOutcome)
  ├── orchestrator.ts  — auto-review state machine (handleAgentEnd → ReviewOutcome)
  │     ├── reviewer.ts (injected as ReviewRunner function)
  │     ├── context.ts  (injected as ContentBuilder function)
  │     ├── architect.ts — prompt + shouldRunArchitectReview
  │     ├── prompt.ts   — assembles the 3-part review prompt
  │     └── changes.ts  — skip decisions (hasFileChanges, isFormattingOnlyTurn)
  ├── commands.ts      — manual review commands (/review N, /review-all, etc.)
  │     ├── reviewer.ts — runReviewSession (direct call)
  │     └── context.ts  — buildPerFileContext
  ├── message-sender.ts — sendReviewResult (formats + sends messages)
  ├── settings.ts      — reads .hardno/settings.json + review-rules.md
  ├── git-roots.ts     — finds git repo roots from file paths
  ├── review-display.ts — animated TUI widget during review
  ├── scaffold.ts      — templates for /scaffold-review-files command
  └── logger.ts        — file logging + structured JSON records

context.ts (content builder)
  ├── helpers.ts       — truncateDiff, clampCommitCount
  ├── ignore.ts        — filters out ignored files
  ├── changes.ts       — buildChangeSummary, collectModifiedPaths
  └── logger.ts
```

## Important patterns


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [inceptionstack/pi-hard-no](https://github.com/inceptionstack/pi-hard-no) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-28 -->
