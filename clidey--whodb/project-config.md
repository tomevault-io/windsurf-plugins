---
trigger: always_on
description: WhoDB is a source-first data management tool. The public GraphQL API and frontend
---

# WhoDB Development Guide

WhoDB is a source-first data management tool. The public GraphQL API and frontend
contract are built around `SourceType`, `SourceContract`, `SourceObject`,
`SourceObjectRef`, and `SourceSessionMetadata`. The current execution layer is
still powered mainly by database plugins under `core/src/plugins/`.

`AGENTS.md` is the canonical agent instruction file for this repository. Tool-
specific files such as `CLAUDE.md` should import or point to this file instead
of duplicating these instructions.

If the `ee/` directory is present, read `ee/AGENTS.md` for additional context. Do not add any code, comments, or references to `ee/` in the CE codebase.

## Non-Negotiable Rules

1. **GraphQL-first** - All new API functionality via GraphQL. Never add HTTP resolvers unless explicitly needed (e.g., file downloads)
2. **No SQL injection** - Never use `fmt.Sprintf` with user input for SQL. Use parameterized queries or GORM builders. See `.agents/docs/sql-security.md`
3. **Plugin architecture** - Never use `switch dbType` or `if dbType ==` in shared code. All database-specific logic goes in plugins. See `.agents/docs/plugin-architecture.md`
4. **Documentation requirements** - All exported Go functions/types need doc comments. All exported TypeScript functions/components need JSDoc. See `.agents/docs/documentation.md`
5. **Localization requirements** - All user-facing strings must use `t()` with YAML keys. No fallback strings. No hardcoded UI text. See `.agents/docs/localization.md`
6. **Verify before completing** - For non-trivial tasks, define success criteria before editing. After finishing, verify: (1) type checks pass (`pnpm run build:ce` for frontend, `go build ./cmd/whodb` for backend), (2) no linting errors, (3) all added code is actually used (no dead code). See `.agents/docs/verification.md`
7. **Show proof** - When making a claim about how something outside of our codebase works, for example a 3rd party library or function, always provide official documentation or the actual code to back that up. Check online if you have to.

## Behavioral Guidelines

These guidelines reduce common LLM coding mistakes. They bias toward caution over speed — for trivial tasks, use judgment.

### 1. Think Before Coding

**Don't assume. Don't hide confusion. Surface tradeoffs.**

Before implementing:
- State your assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them — don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.

### 2. Simplicity First

**Minimum code that solves the problem. Nothing speculative.**

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- No fallback logic unless explicitly asked — if you think it's needed, ask first.
- No defensive programming — if an edge case needs handling, ask first.
- If you write 200 lines and it could be 50, rewrite it.

Ask yourself: "Would a senior engineer say this is overcomplicated?" If yes, simplify.

### 3. Surgical Changes

**Touch only what you must. Clean up only your own mess.**

When editing existing code:
- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- If you notice unrelated dead code, mention it — don't delete it.

When your changes create orphans:
- Remove imports/variables/functions that YOUR changes made unused.
- Don't remove pre-existing dead code unless asked.

The test: Every changed line should trace directly to the user's request.

### 4. Goal-Driven Execution

**Define success criteria. Loop until verified.**

Transform tasks into verifiable goals:
- "Add validation" → "Write tests for invalid inputs, then make them pass"
- "Fix the bug" → "Write a test that reproduces it, then make it pass"
- "Refactor X" → "Ensure tests pass before and after"

For multi-step tasks, state a brief plan:
```
1. [Step] → verify: [check]
2. [Step] → verify: [check]
3. [Step] → verify: [check]
```

Strong success criteria let you loop independently. Weak criteria ("make it work") require constant clarification.

## Execution Workflow

For non-trivial tasks, use a short goal-driven loop:

1. Identify the expected behavior or failure path.
2. Choose the smallest change that satisfies the request.
3. Add or update focused tests when the behavior is testable and the risk justifies it.
4. Run the relevant verification commands and inspect the diff before finishing.

## Agent Operating Model

- Treat `AGENTS.md` as the shared source of truth for Codex, Claude Code via import, opencode, Pi, and other compatible coding agents.
- Use `.agents/README.md` as the index for deeper agent guidance, and read only the relevant linked file before editing.
- Keep always-loaded instructions concise. Move detailed workflows, checklists, and runbooks into linked docs rather than duplicating them in tool-specific files.
- Use planning mode for multi-file, risky, architectural, or ambiguous changes. Skip formal plans for obvious single-purpose edits.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [clidey/whodb](https://github.com/clidey/whodb) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->
