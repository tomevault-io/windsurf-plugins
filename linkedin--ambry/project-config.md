---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

---

## 1. LLM Behavioral Guidelines

Behavioral guidelines to reduce common LLM coding mistakes. Merge with project-specific instructions as needed.

**Tradeoff:** These guidelines bias toward caution over speed. For trivial tasks, use judgment.

### Think Before Coding

**Don't assume. Don't hide confusion. Surface tradeoffs.**

Before implementing:

- State your assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them — don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.

Storage systems punish assumptions. Read the relevant source files, understand the data flow, and trace the call path before proposing changes. If you are unsure about a component's behavior, read its implementation and tests before modifying it.

### Simplicity First

**Minimum code that solves the problem. Nothing speculative.**

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- If you write 200 lines and it could be 50, rewrite it.

Ask yourself: "Would a senior engineer say this is overcomplicated?" If yes, simplify.

### Surgical Changes

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

### Goal-Driven Execution

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

**These guidelines are working if:** fewer unnecessary changes in diffs, fewer rewrites due to overcomplication, and clarifying questions come before implementation rather than after mistakes.

**Use subagents by default** where possible for parallel research, exploration, and independent subtasks.

---

## 2. PR Guidelines

PRs must be logically isolated and easy to review: small, coherent diffs with one purpose per PR. Avoid drive-by refactors.

PR descriptions **must** include a `## Testing Done` section (not `## Testing`).

Aim for as close to 100% unit test coverage as practical for touched logic. Cover risky paths and edge cases.

---

## 3. Ambry System-Level Invariants

### Durability Paranoia

Ambry is a production blob storage system. **"You must never lose data"** is a non-negotiable invariant. Every PR must include a durability risk analysis and tests/mitigations.

**Tradeoff priority order** (when forced to choose):
1. **Durability** — Data must never be lost or silently corrupted
2. **Security** — ACLs, encryption, and auth must not be weakened
3. **Availability** — The system must remain serving
4. **Operability** — Changes must be observable and debuggable
5. **Efficiency** — Resource usage matters but not at the cost of the above
6. **Performance** — Latency and throughput matter but not at the cost of the above

No repo-specific guideline can override the durability invariant.

### Engineering Principles

- Explicit invariants and clear failure-mode handling
- Safe retries and idempotency — operations must be safe to retry after partial failure
- Defensive coding — validate inputs at system boundaries, handle nulls explicitly
- Code should be readable by humans and LLM agents (future maintainers)
- Strong bias toward correctness and operability over cleverness

### Durability Risk Checklist

Apply to every PR. If any item is checked, the PR requires extra scrutiny and explicit testing.

- [ ] **Write path** — Does this change touch PUT, named blob PUT, TTL update, or delete operations?
- [ ] **Metadata** — Does this change modify how blob metadata is stored, read, or indexed?
- [ ] **Ordering / Atomicity** — Could operations complete partially, leaving inconsistent state?
- [ ] **Callback semantics** — Could success be reported to a client before data is durable?
- [ ] **Resource cleanup** — Does this modify ByteBuf release, Closeable.close(), or stream lifecycle?
- [ ] **Retries / Idempotency** — Are retries safe? Could a retry cause duplicate writes or skipped deletes?
- [ ] **Partial failures** — What happens if this operation fails midway? Is the system left in a recoverable state?

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [linkedin/ambry](https://github.com/linkedin/ambry) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
