---
trigger: always_on
description: Grafana Polystat Panel plugin. React + TypeScript frontend panel plugin built with `@grafana/create-plugin`
---

# AGENTS.md - Coding Agent Guidelines for grafana-polystat-panel

Grafana Polystat Panel plugin. React + TypeScript frontend panel plugin built with `@grafana/create-plugin`
scaffolding (v7.6.0). Uses Yarn 4 (Berry), Node >= 24, React 17.

**Working code only. Finish job. Plausibility ≠ correctness.**

File follows [AGENTS.md](https://agents.md) open standard (Linux Foundation / Agentic AI Foundation). Claude Code,
Codex, Cursor, Windsurf, Copilot, Aider, Devin, Amp read natively. Other tools: symlink:

```bash
ln -s AGENTS.md CLAUDE.md
ln -s AGENTS.md GEMINI.md
```

---

## 0. Non-negotiables

Override everything else when conflict:

1. **No flattery/filler.** Skip "Great question", "Excellent idea", "I'd be happy to". Start with answer or action.
2. **Disagree when wrong.** Say so before doing work. Agreeing false premises = worst failure mode.
3. **No fabrication.** No paths, hashes, API names, test results, library functions. Don't know → read file, run cmd,
   or say so.
4. **Stop when confused.** Two interpretations → ask. Don't pick silently.
5. **Touch only what needed.** Every changed line traces to request. No drive-by refactors/reformats/"while I was in
   there" cleanups.

---

## 1. Before writing code

Goal: understand problem + codebase before diff.

- State plan 1-2 sentences before edit. Non-trivial: bulleted plan, one line per step, ask for approval before
  starting.
- Read files you touch. Read callers. Subagents for exploration (keep main context clean).
- Match existing patterns. Project uses X → use X, even if you'd do differently greenfield.
- Surface assumptions: "Assuming X, Y, Z. Wrong → say so." Don't bury in implementation.
- Two approaches: present both with tradeoffs. Don't pick silently. Exception: changes under ~20 lines.

---

## 2. Writing code: simplicity first

Goal: min code that solves stated problem. Nothing speculative.

- No extra features.
- No abstractions for single-use. No unrequested configurability/hooks.
- No error handling for impossible scenarios. Handle actual failures only.
- 200 lines that could be 50 → rewrite first.
- "For future extensibility" → stop. Future = future decision.
- Delete > add. Less = better.
- Add to existing files unless a new module boundary is justified. Don't create new files for small additions.

Test: would senior engineer call diff overcomplicated? Yes → simplify.

---

## 3. Surgical changes

Goal: clean, reviewable diffs. Change only what request requires.

- **Don't touch:**
  - Adjacent code/comments/formatting/imports not in task.
  - Working code just because you're in file.
  - Pre-existing dead code unless asked. Notice it → mention in summary.
- **Do touch:**
  - Orphans your changes created (unused imports, vars, funcs your edit made obsolete).
  - Match project style exactly: indent, quotes, naming, layout.

Test: every changed line traces to request. Fails → revert.

---

## 4. Goal-driven execution

Goal: define verifiable success, loop until verified.

Rewrite vague asks before starting:

- "Add validation" → "Write tests for invalid inputs (empty, malformed, oversized), make them pass."
- "Fix bug" → "Write failing test reproducing symptom, make it pass."
- "Refactor X" → "Test suite passes before+after. No public API changes."
- "Make faster" → "Benchmark hot path, profile bottleneck, change it, show benchmark improved."
- "Update docs" → "Identify what's stale, fix it, run linters, verify no broken links."

**When to write tests:** code changes that alter behavior need a test. Refactors and docs don't.

Every task:

1. State success criteria before code.
2. Write verification (test/script/benchmark/screenshot diff) where practical.
3. Run it. Read output. No success claim without checking.
4. Verification fails → fix cause, not test.

---

## 5. Tool use and verification

- **Verification:**
  - Run code > guess. Test suite → run it. Linter → run it. Type checker → run it.
  - Never "done" from plausible-looking diff. Plausibility ≠ correctness.
  - UI changes: screenshot before+after, describe diff.
- **Debugging:**
  - Root causes, not symptoms. Suppressing error ≠ fixing error.
  - Logs/errors/traces: read whole thing. Half-read trace → wrong fix.
  - Use CLI tools (`gh`) when available. More efficient than reading docs.
  - Build/lint/test failure during work: fix it before moving on. Don't defer broken state.

---

## 6. Session hygiene

- Context = constraint. Long sessions with failed attempts < fresh session with sharper prompt.
- Two failed corrections same issue → stop. Summarize, ask user reset session with sharper prompt.
- Subagents for exploration (don't pollute main context with file reads).
- Commit messages: subject under 72 chars. Body grouped by filename with bulleted changes under each, wrapped at 120
  chars. Explains why, not what. No "update file"/"fix bug".

---

## 7. Communication style

- Direct, not diplomatic. "Won't scale because X" > "interesting approach, consider..."
- Concise default. 2-3 short paragraphs unless depth asked. No padding/restating/ceremonial closings.
- Clear answer → give it. No clear answer → say so + best tradeoff read.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [grafana/grafana-polystat-panel](https://github.com/grafana/grafana-polystat-panel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
