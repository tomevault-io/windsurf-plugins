---
trigger: always_on
description: These rules override all other instructions when they conflict. They exist to reduce LLM coding drift, over-engineering, and speculative changes.
---

# AGENTS.md

## Agent Behavioral Contract

These rules override all other instructions when they conflict. They exist to reduce LLM coding drift, over-engineering, and speculative changes.

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

## Agent Execution Protocol

Operational execution rules for this coding agent session.

### 5. Operational, Not Conversational

Work from explicit instructions. If the user's request is vague, stop and clarify. Do not proceed on best-guess intent.

**Pattern:** `read-before-write → evidence-before-action → minimal diff → verify-before-report`

### 6. Read Before Write

Do not infer repository paths, APIs, helpers, or behavior. Confirm facts by:
1. Reading files explicitly (`read`, `grep`, `find_files`).
2. Following local project docs (`AGENTS.md`, `README.md`, `docs/adr/`, `docs/`).
3. Running verification commands before claiming understanding.

### 7. Lock the Scope

- No opportunistic refactors.
- No unrelated cleanup.
- No files outside the target set.
- If the user asks for X, deliver X. Do not also deliver Y because "it might be useful."

### 8. Define Stop Gates

Ambiguity, missing files, conflicting docs, forbidden commands, or unclear scope should produce **BLOCKED**. Stop and ask before proceeding.

Do not proceed with best-guess assumptions. Surface uncertainty explicitly.

### 9. Require Proof, Not Confidence

Confirm work with actual checks before claiming success:
- Run tests (`npm run check`, domain-specific suites).
- Check logs or command output.
- Verify file contents match intent.
- Re-read your own edits to ensure correctness.

### 10. Compaction Recovery

For long tasks, recovery state must be inspectable:
- Use `git diff` to show current changes.
- Track modified files explicitly.
- Persist verification state.
- Keep a running result artifact if the task spans multiple turns.

## Project mission

Build a Pi extension that gives Pi durable long-term memory through Hindsight.

The extension must be:

- technically aligned with official Hindsight best practices
- idiomatic to Pi’s extension/package model
- reliable under outages
- easy to inspect, debug, and extend
- safe to use as the base for future Pi extensions

## Source of truth

### Technical source order

1. Official Hindsight docs and API behavior
2. Official Pi `pi-mono` extension/session/package docs
3. This repository’s PRD, ADRs, GitHub issues, and current docs
4. Public reference repos only as implementation inspiration
5. User notes/gists only as hypotheses, never as authority

### Use these constraints

- Use the official Hindsight TypeScript client
- Treat Hindsight docs and OpenAPI behavior as source of truth
- Treat Pi’s documented extension lifecycle as source of truth
- Do not invent undocumented Pi internals or undocumented Hindsight request shapes

### Shared contributor guidance

Human and agent guidance must stay aligned. When changing source-of-truth order, contributor workflow, verification expectations, memory policy, or definition-of-done criteria, update `CONTRIBUTING.md` and this file together. `CONTEXT.md` is the shared vocabulary for both human-facing and agent-facing docs.

### Work tracking


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [luxus/pi-hindsight](https://github.com/luxus/pi-hindsight) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
