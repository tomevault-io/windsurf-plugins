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

Make the codebase legible to agents. The work to make a codebase legible to an agent — written-down
conventions, skills, rules, intent docs — is simply the debt you owe to your human engineers; every
entry pays it down for both audiences at once.

---

## Getting Started

1. **Read this file** — it's the baseline for how we work
2. **Use your tools naturally** — read files, search code, run builds directly
3. **Delegate when triggered** — see delegation rules in Claude Code's CLAUDE-FULL.md. Multi-file exploration, security-sensitive code, and test writing MUST go to agents; don't reason your way out of it.
4. **Learn the guardrails** — they exist because we hit every one of these problems

Don't over-engineer your workflow. Start simple, add complexity only when you feel friction.

---

## Response Calibration

Claude Opus 4.8 calibrates response length to task complexity rather than defaulting to fixed verbosity. Match your output to what was asked:

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

**Positive guidance beats negative**: state how to respond (concise, direct) rather than what to avoid. 4.8 interprets prompts literally — a rule like "don't be verbose" is weaker than "respond in 1-2 sentences for simple questions."

---

## Guardrails

These rules exist because we've seen them violated repeatedly. Non-negotiable.

### Laziness Ladder (Before Writing Code)
The best code is the code you don't write. Before generating anything, stop at the **first rung that holds**:

1. **Does this need to exist?** — if no, skip it (YAGNI). Question the request before solving it.
2. **Does the standard library / runtime already do this?** — use it.
3. **Does a native platform feature cover it?** — use it.
4. **Does an already-installed dependency solve it?** — use it; don't add a new one.
5. **Can it be one line?** — make it one line.
6. **Only then** — write the minimum that works.

Default to deletion over addition, boring over clever, fewest files possible. No abstractions, dependencies, or boilerplate nobody asked for. When two stdlib approaches tie on size, pick the edge-case-correct one.

**Lazy, not negligent.** The ladder never applies to trust-boundary/input validation, error handling that prevents data loss, security, accessibility, or anything explicitly requested — those are always built in full. It also bends for real-world physical constraints (hardware drift, sensor inaccuracy) when the task involves them.

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

### Completeness Is Cheap
AI-assisted coding pushes the marginal cost of finishing toward zero. When the complete version of the thing you're **already building** costs minutes more than the shortcut, do the complete thing — every edge case, error path, and test. "Ship the 90%, defer the rest" is legacy thinking from when human typing was the bottleneck.

This is bounded by scope, not a license to expand it. Complete the **unit you're deliberately touching**; it does not override `Bug Fix Scope` (a fix stays minimal) or `Surface Conflicts`. Finishing a bounded module is a "lake" — boil it. Rewriting an adjacent system is an "ocean" — flag it as out of scope, don't start it.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [darkroomengineering/cc-settings](https://github.com/darkroomengineering/cc-settings) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-20 -->
