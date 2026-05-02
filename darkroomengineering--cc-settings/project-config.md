---
trigger: always_on
description: > Coding standards and guardrails for AI-assisted development.
---

# Darkroom Engineering

> Coding standards and guardrails for AI-assisted development.
> Works with Claude Code, Codex, Cursor, Copilot, Windsurf, and any AGENTS.md-compatible tool.

---

## Philosophy

This codebase will outlive any single contributor. Every shortcut becomes someone else's burden.
Fight entropy. Leave the codebase better than you found it.

---

## Getting Started

1. **Read this file** — it's the baseline for how we work
2. **Use your tools naturally** — read files, search code, run builds directly
3. **Delegate when triggered** — see delegation rules in Claude Code's CLAUDE-FULL.md. Multi-file exploration, security-sensitive code, and test writing MUST go to agents; don't reason your way out of it.
4. **Learn the guardrails** — they exist because we hit every one of these problems

Don't over-engineer your workflow. Start simple, add complexity only when you feel friction.

---

## Response Calibration

Claude Opus 4.7 calibrates response length to task complexity rather than defaulting to fixed verbosity. Match your output to what was asked:

- **Simple questions** (1-2 sentences): direct answer, no preamble, no trailing summary
- **Lookups ("where is X", "what does Y do")**: one short paragraph + file:line reference
- **Single-file changes**: brief description + show the diff. Don't restate what the diff already shows
- **Multi-file or complex work**: brief plan → execute → 1-2 sentence summary of what changed
- **Explanations**: use headers/bullets only when structure genuinely helps; avoid ceremonial formatting for short answers

**Never**:
- Add trailing summaries that restate the diff
- Explain what you're about to do before every tool call (one sentence before a batch is enough)
- Use "I'll now..." / "Let me..." preambles; just do the thing and announce completion
- Pad responses with caveats or qualifications that don't change the outcome

**Positive guidance beats negative**: state how to respond (concise, direct) rather than what to avoid. 4.7 interprets prompts literally — a rule like "don't be verbose" is weaker than "respond in 1-2 sentences for simple questions."

---

## Guardrails

These rules exist because we've seen them violated repeatedly. Non-negotiable.

### Read Before Edit
**Never change code you haven't read.** Research the codebase before editing — open the file, trace the callers, understand the context. Edit-first behavior produces shallow fixes and regressions. If you're about to modify something you haven't read in this session, stop and read it first.

### 2-Iteration Limit
If an approach fails after **2 attempts**, STOP:
1. Summarize what you tried and why it failed
2. Present **2-3 alternative approaches** with trade-offs
3. Ask which direction to take

Never burn 6+ attempts on the same strategy. Fail fast, pivot deliberately.

### Bug Fix Scope
When fixing a bug, stay **confined to files directly related to the bug**:
- Don't refactor adjacent code "while you're in there"
- Don't upgrade dependencies as part of a bug fix
- Don't touch files outside the immediate blast radius
- A bug fix PR should be reviewable in under 2 minutes

### Verify After Every Fix
Run the build after any fix and verify it passes **before moving on**. Never stack untested fixes — cascading errors eat context and compound regressions.

### Pre-Commit Verification
Before ANY commit:
1. Run type checking (`tsc --noEmit` for TypeScript) — fix all errors
2. Run the build command — fix all errors
3. Run existing tests — fix all failures

**Never commit code that doesn't build.**

### Never Fake Measurements
NEVER fabricate output from Lighthouse, bundle size tools, performance profilers, test runners, or build systems. If you can't run a tool, say so.

### Visual/Spatial Honesty
For sub-pixel rendering, WebGL, physics, complex animations, or canvas — acknowledge limitations upfront. Provide best-effort with clear TODOs, and suggest the user validate visually.

For CSS/visual bugs: if a fix doesn't work after 2 attempts, propose **3 fundamentally different approaches** and let the user pick.

### Post-Compaction Recovery
After any compaction or context reset, **before continuing work**:
1. Re-read the task plan (todo, plan file, or issue)
2. Re-read the files you're actively modifying
3. Run `git diff --stat` to see what's changed
4. Only then continue implementation

Never assume you remember file contents or task state after compaction. Context loss is silent — re-read, don't guess.

### Neutral Exploration
When investigating code (auditing, reviewing, exploring), use **neutral prompts** that don't bias toward a specific outcome:
- Say "analyze the logic and report all findings" — not "find the bug"
- Say "review the auth flow and describe what happens" — not "what's wrong with auth"
- Say "trace the data flow and report" — not "where's the data leak"

Biased prompts cause agents to manufacture issues that don't exist.

### TODO Comments Are Instructions
When you encounter a `TODO`, `FIXME`, or `HACK` comment, **implement it** — don't delete it. Removing a TODO without doing the work is marking your own homework complete by erasing the assignment.

### Plan Before Multi-File Changes
Before any change touching **5+ files**, outline the plan first:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [darkroomengineering/cc-settings](https://github.com/darkroomengineering/cc-settings) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
