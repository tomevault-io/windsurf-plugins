---
trigger: always_on
description: ACEvoPerf is a `dstorage.dll` proxy mod for Assetto Corsa EVO plus Python tools. Read
---

# CLAUDE.md

## 0. Project Rules

ACEvoPerf is a `dstorage.dll` proxy mod for Assetto Corsa EVO plus Python tools. Read
`.agent/INDEX.md` first.

- Gates before any review: `build.ps1` completes with zero errors, and a game launch writes an
  `acevo_perf.log` that shows the changed behaviour. Tool changes run against the real package or
  settings file.
- Tooling: PowerShell for `build.ps1` and `release.ps1`, `py -3` for `tools/*.py` on the owner's
  machine, `python3` in Git Bash. The game folder is only ever the `ACEVO_GAME_DIR` environment
  variable or an explicit option, never a path written into code, scripts or docs.
- Installing is copying the zip payload into the game folder, never a script. The game must be
  closed before `dstorage.dll` can be replaced.
- Runtime output (`acevo_perf*.log`, `*.csv`, game logs) is never committed. Analysis sessions
  live in `logs/<session>/` inside the repo (gitignored) and are summarised into
  `.agent/docs/research/`.
- Source layout: headers under `include/acevo/<area>/`, sources under `src/<area>/`, one folder
  per concern, `src/dllmain.cpp` only wires them. Python tools in `tools/`, their data files in
  `tools/data/`.
- `README.md`, `CHANGELOG.md` and `dist/README.txt` are written for players who have never seen the
  code, in short plain lines without internals or measurements. Read
  `.agent/docs/ops/public-docs.md` before changing any of them.
- `CHANGELOG.md` at the root records every user visible change in the same commit, one line in the
  player's words, under the next version's heading marked `(unreleased)` until that version is cut.
- Areas used by the trackers: `streaming`, `render`, `ui`, `stability`, `engine-flags`,
  `foundation`, `tooling`, `release`, `docs`.

## 1. Coding Guideline

Behavioral guidelines to reduce common LLM coding mistakes. Merge with project-specific instructions as needed.

> **Tradeoff:** These guidelines bias toward caution over speed. For trivial tasks, use judgment.

### 1.1 Think Before Coding

**Don't assume. Don't hide confusion. Surface tradeoffs.**

Before implementing:

- State your assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them, don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.

### 1.2 Simplicity First

**Minimum code that solves the problem. Nothing speculative.**

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- If you write 200 lines and it could be 50, rewrite it.

Ask yourself: "Would a senior engineer say this is overcomplicated?" If yes, simplify.

### 1.3 Surgical Changes

**Touch only what you must. Clean up only your own mess.**

When editing existing code:

- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- If you notice unrelated dead code, mention it, don't delete it.

When your changes create orphans:

- Remove imports, variables, and functions that YOUR changes made unused.
- Don't remove pre-existing dead code unless asked.

The test: every changed line should trace directly to the user's request.

### 1.4 Goal-Driven Execution

**Define success criteria. Loop until verified.**

Transform tasks into verifiable goals:

- "Add validation" → "Write tests for invalid inputs, then make them pass"
- "Fix the bug" → "Write a test that reproduces it, then make it pass"
- "Refactor X" → "Ensure tests pass before and after"

For multi-step tasks, state a brief plan:

```text
1. [Step] → verify: [check]
2. [Step] → verify: [check]
3. [Step] → verify: [check]
```

Strong success criteria let you loop independently. Weak criteria ("make it work") require constant clarification.

### 1.5 Tests That Can Fail

**Write tests that can fail. Make the code pass them, not the other way around.**

When you write tests:

- A test that passes no matter what the code does is testing nothing. Write it so it fails first, which proves it is actually checking something, then make the code turn it green.
- Don't write tests that already pass against the current code. If a test is green before you change the implementation, it isn't exercising the behavior you came to add.
- Make the code pass the test, not the test pass the code. Loosening an assertion to match a wrong output is faking the result.
- The only reason to touch a test is that the test itself is wrong about the expected behavior. When that happens, say so and state what the correct expectation is.

> **These guidelines are working if:** fewer unnecessary changes in diffs, fewer rewrites due to overcomplication, and clarifying questions come before implementation rather than after mistakes.
---

## 2. Coding Style


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ecstra/ACEvoPerf](https://github.com/ecstra/ACEvoPerf) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-18 -->
