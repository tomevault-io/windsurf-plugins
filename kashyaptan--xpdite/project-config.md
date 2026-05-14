---
trigger: always_on
description: Xpdite is an **always-on-top Electron desktop app** that wraps a React UI and a Python FastAPI backend to deliver an AI chat assistant with screenshot OCR, MCP tool calling, and multi-provider LLM support (Ollama, Anthropic, OpenAI, Gemini, OpenRouter). Python dependencies are managed with **UV**; frontend with **Bun**.
---

# Xpdite — CLAUDE.md

Xpdite is an **always-on-top Electron desktop app** that wraps a React UI and a Python FastAPI backend to deliver an AI chat assistant with screenshot OCR, MCP tool calling, and multi-provider LLM support (Ollama, Anthropic, OpenAI, Gemini, OpenRouter). Python dependencies are managed with **UV**; frontend with **Bun**.

---

## Workflow

### Read More Than Less
Always read all relevant and connected files before writing new code. It is always better to over-read than to miss context.

### Freedom and Direction
You are extremely knowledgeable — don't be afraid to use that. If you have concerns, suggestions, or improvements, raise them. Discussion and clarification lead to the best possible outcome.

### Planning
Enter plan mode for non-trivial tasks. Get the correct info and details before executing. For trivial tasks this is unnecessary — don't over-engineer.

### Sub-agents for Information Gathering
Spawn as many sub-agents as you need **in parallel** for any read-only task that just needs a result — reading files, searching for patterns, exploring the directory structure, checking how something is implemented. The goal is to keep the main context window clean and focused. Do NOT use sub-agents when the reasoning process itself is needed in the main context.

### Sub-agents for Self-Review — Best-of-N + Parallel + De-dup

Use the multi-agent review pipeline for high-risk code changes (new services, migrations, auth/security-sensitive paths, complex concurrency changes). For routine edits, direct implementation plus targeted validation is sufficient. If the user explicitly asks to skip the review pipeline for a task, skip it.

---

#### Stage 1 — Parallel Focused Reviewers (run all three simultaneously)

Spawn **three independent sub-agent reviewers in parallel**, each with a *single narrow focus*. Provide each agent the changed files + the `CODE_REVIEW_GUIDE.md`. Give each a different lens:

| Agent | Focus | Checklist Phases |
|---|---|---|
| **Reviewer A — Correctness & Logic** | Logic bugs, edge cases, async errors, wrong return types, mutation bugs | Phase 1 (Correctness) |
| **Reviewer B — Security & Resilience** | Injection, hardcoded secrets, missing auth, unhandled errors, resource leaks, timeouts | Phase 2 (Security) + Phase 3 (Error Handling) |
| **Reviewer C — Performance & Quality** | N+1 queries, O(n²) loops, dead code, naming, complexity, missing tests | Phase 4 (Performance) + Phase 5–7 (Simplification, Style, Testability) |

Each reviewer produces a raw findings list in the `CODE_REVIEW_GUIDE.md` report format. They work completely independently and must **not** see each other's output.

---

#### Stage 2 — Best-of-N Synthesis (optional but recommended for large diffs)

For large or high-risk changes (new services, DB migrations, auth flows), spawn **two additional Reviewer A agents** (correctness is the highest-value check) and pick the best / most complete findings across all three correctness reports. This is "best of N" — independent runs of the same task surface different bugs.

---

#### Stage 3 — De-dup Judge Agent

Spawn a **single judge sub-agent** that receives all raw reports from Stage 1 (and Stage 2 if run). The judge must:

1. **Merge** all findings into a single ranked list (Critical → High → Medium → Low).
2. **De-duplicate** — if multiple reviewers flag the same issue, keep one entry with a note that N reviewers flagged it (higher confidence).
3. **Resolve contradictions** — if Reviewer A says something is fine and Reviewer B flags it, the judge investigates and decides.
4. **Filter false positives** — mark any finding as a false positive if it flags intentional, correct behavior (e.g. flagging `check_same_thread=False` on SQLite as a bug when it's required by our DB pattern).
5. **Produce the final `CODE_REVIEW_GUIDE.md` report** with a clear Production Readiness Verdict.

---

#### Stage 4 — Fix & Verify

- Incorporate all Critical and High findings before responding.
- Fix all problems found Critical first, then High, then medium, then low.
- If any findings require human review, flag it and mention in review document.
- After fixes, spawn a **final lightweight verification agent** to confirm the fixes are correct and didn't introduce regressions.
- Make sure to create a code_review_[topic of review].md file in the code_review folder after the review stating all the problems found and how they were fixed.

---

### Post-Review Action
Read the Testing section and determine if new tests are needed based on the changes made. Once the entire task is complete, update any relevant CLAUDE and documentation files to reflect the changes.
After every code implementation, run all lint checks and tests and fix all issues before considering the task complete.
---

## Dev Commands

```bash
bun run dev              # start everything: React (Vite), Electron, Python server, Ollama (GPU via scripts/start-ollama.mjs)
bun run dev:react        # Vite only (port 5123)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [KashyapTan/Xpdite](https://github.com/KashyapTan/Xpdite) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
