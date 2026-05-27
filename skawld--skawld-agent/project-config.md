---
trigger: always_on
description: This repository will contain **skawld**, an open-source TypeScript agent harness for software-engineering tasks (think: a Claude Code-like loop, with our own modifications). Right now the repository only contains a `docs/` folder. Your job is to read those docs and implement the system they describe.
---

# AGENTS.md

This repository will contain **skawld**, an open-source TypeScript agent harness for software-engineering tasks (think: a Claude Code-like loop, with our own modifications). Right now the repository only contains a `docs/` folder. Your job is to read those docs and implement the system they describe.

## Behavioral guidelines

Behavioral rules to reduce common LLM coding mistakes. These bias toward caution over speed; for trivial tasks, use judgment.

### 1. Think before coding

**Don't assume. Don't hide confusion. Surface tradeoffs.**

Before implementing:
- State your assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them — don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.

### 2. Simplicity first

**Minimum code that solves the problem. Nothing speculative.**

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- If you write 200 lines and it could be 50, rewrite it.

Ask yourself: "Would a senior engineer say this is overcomplicated?" If yes, simplify.

### 3. Surgical changes

**Touch only what you must. Clean up only your own mess.**

When editing existing code:
- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- If you notice unrelated dead code, mention it — don't delete it.

When your changes create orphans:
- Remove imports/variables/functions that YOUR changes made unused.
- Don't remove pre-existing dead code unless asked.

The test: every changed line should trace directly to the user's request.

### 4. Goal-driven execution

**Define success criteria. Loop until verified.**

Transform tasks into verifiable goals:
- "Add validation" → "Write tests for invalid inputs, then make them pass."
- "Fix the bug" → "Write a test that reproduces it, then make it pass."
- "Refactor X" → "Ensure tests pass before and after."

For multi-step tasks, state a brief plan:

```
1. [step] → verify: [check]
2. [step] → verify: [check]
3. [step] → verify: [check]
```

Strong success criteria let you loop independently. Weak criteria ("make it work") require constant clarification.

### 5. Test after every change

**No change is "done" until tests run green.**

After any code change — even a one-line edit — run the relevant tests before reporting back:

- After editing a file: `bun test path/to/file.test.ts` (or the whole suite if scope is unclear).
- After finishing a module: `bun run typecheck && bun test && bun run build`.
- After changing build/config: full smoke (`bun install`, `bun run typecheck`, `bun run build`, `bun test`).
- If you cannot test (UI/runtime not reachable from here): say so explicitly. Do not claim success.

Type checking and build success verify code correctness, not feature correctness. Run the tests.

If tests fail: fix the cause, don't ignore, skip, or weaken the test. Never `--no-verify`, `.only`, `.skip`, or weaken assertions to pass.

---

**These guidelines are working if:** fewer unnecessary changes in diffs, fewer rewrites due to overcomplication, and clarifying questions come before implementation rather than after mistakes.

## How to use this repository

1. **Start at `docs/index.html`.** It is the single entry point. It explains what skawld is, the overall architecture, the module map, the reading order, and the project glossary.
2. **Follow the reading order in `docs/index.html`.** The module docs are numbered (`00-` through `10-`). Earlier docs are foundational for later ones.
3. **Treat every doc as authoritative.** Type names, event names, error names, field names, and behaviors are normative. Do not rename or restructure them without an explicit instruction from the user.
4. **Each module doc has a "Build checklist" near the end.** Use it to track what you have implemented and what remains. Mark items off as you go.
5. **When in doubt, re-read the relevant doc.** If a doc seems ambiguous, surface the ambiguity to the user before guessing.

## What lives where

```
.
├── AGENTS.md            ← you are here
├── docs/                ← read this folder first
│   ├── index.html       ← start here
│   ├── styles.css
│   ├── 00-foundations.html
│   ├── 01-providers.html
│   ├── 02-tools.html
│   ├── 03-permissions.html
│   ├── 04-sessions.html
│   ├── 05-agent-loop.html
│   ├── 06-system-prompt.html
│   ├── 07-sdk.html
│   ├── 08-cli.html
│   ├── 09-config.html
│   └── 10-project-setup.html
└── (src/ etc. will be created by you as you implement)
```

## Hard rules

- **TypeScript only** for the core. ESM. Bun 1.1+ (Bun-native — no Node runtime support). `bun build` for JS, `tsc --emitDeclarationOnly` for `.d.ts`, `bun test` for tests, `bun:sqlite` for the session store.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [skawld/skawld-agent](https://github.com/skawld/skawld-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-27 -->
