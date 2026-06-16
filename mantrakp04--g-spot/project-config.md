---
trigger: always_on
description: `CLAUDE.md` is a symlink to this file.
---

# AGENTS.md

`CLAUDE.md` is a symlink to this file.

## Workflow capture (meta-rule — read every message through this)

This file is the user's living workflow. On **every** incoming message, before acting, classify the intent:

- **One-time / ad-hoc** (specific to this task, this file, this moment) → just do it. Don't touch this file.
- **Generic / recurring** (a preference, habit, or rule that would apply again to similar future tasks) → fold it into the `Workflow` section below, then carry it out.

Heuristics for "generic enough to capture":
- It's phrased as a general rule ("always…", "whenever…", "from now on…", "I want X to happen when Y").
- It's a process/verification step that isn't tied to one specific file or one-off bug.
- You'd plausibly redo it across multiple unrelated tasks (e.g. "verify UI changes in a browser when you make a UI change").

When capturing:
- Add or update the relevant bullet under `Workflow` (don't duplicate — merge into an existing rule if it overlaps).
- Keep it terse and actionable, matching the style of the rest of this file.
- Do it silently as part of the task; a one-line mention that you updated the workflow is enough.

## Workflow

Recurring conventions captured from the user. Follow these on matching tasks:

- **Verify UI in a browser.** Whenever a change touches the UI, run the app and confirm the change visually in a browser (use a browser/QA skill) before calling it done. The dev server with HMR runs on **port 3002** (`apps/web`, `http://localhost:3002`) — not 3001 (3001 is `apps/server`). Ports are derived from `packages/env/src/dev-ports.ts` (prefix `30`: server `3001`, web `3002`, landing `3003`, relay `3004`).
- **Simplicity over forcing the current arch.** Put logic where it's simplest and best, not where the existing structure happens to sit. If something is cleaner/easier on the server, move it server-side; if it belongs on the client, move it there. Don't contort code to fit the current architecture — rewrite it for the side that makes it simpler, and clean up the old side (rename/remove stale paths) when you do.

## Task completion requirements

### Type checking

```
// MANDATORY RUN at the end
bun check-types
```

Always run `bun check-types` after code changes before finalizing. If it fails, re-run until it passes. Turbo runs `check-types` in each workspace that defines it.

## Communication style

**Be concise and direct. No fluff. Match the energy.**

User uses casual language ("bro", "dawg", "ugh"). Keep responses terse and actionable. When something breaks, diagnose fast, fix faster.

### Handling interruptions

When a new message arrives mid-task, **don't drop what you're doing by default.** Triage it:

1. **Additive context** (extra detail, clarification, "also do X") — absorb it into the current task and keep going.
2. **Correction / "wait, that's wrong"** — stop the current step, address the correction, then resume or adjust course.
3. **New unrelated task** — finish the current task first, then start the new one. Don't context-switch mid-implementation.
4. **Explicit hard stop** ("stop", "drop this", "do X instead") — respect it immediately and switch.
5. **Urgent/blocking** ("you're breaking X", "that file is wrong") — prioritize now, but come back and finish the original task if it still makes sense.

Rule of thumb: **stay on task unless told otherwise or the interrupt would make continued work harmful/wasteful.**

### Clarification gate (mandatory)

- Before implementing any non-trivial change, ask for confirmation if scope/intent is not explicitly specified.
- Do not assume architectural behavior for stateful flows (worker lifecycle, mode switching, persistence, auth propagation, background execution).
- If multiple reasonable implementations exist, present options briefly and wait for selection before coding.
- When a change can affect cross-context behavior (chat vs workflow, server vs client, trigger vs interactive path), ask first and get approval.
- Default to asking one targeted clarifying question rather than executing on inferred intent.

## Research grounding (web search)

**Treat the public web as the default source of truth for anything outside this repository.** Training data is not sufficient on its own for research, comparisons, or “how does X work today?”

- **Run web search** before you assert facts about third-party APIs, CLI behavior, framework versions, platform limits, security advisories, deprecations, licensing, or current events. Prefer primary sources (official docs, release notes, standards bodies) surfaced via search.
- **Ground questions to the user** in what search already showed. Prefer “I found [A] vs [B] in current docs; which matches your setup?” over guesses. If search is inconclusive, say that and point to what you checked.
- **Internal-only work** (reading this repo, inferring types, following existing patterns) does not require web search. **External claims and integration decisions do.**
- **Stack with other tools:** **DeepWiki** (or repo docs) for a specific GitHub project’s behavior; **web search** for freshness, version matrices, and anything not covered by those. If two sources disagree, trust newer primary documentation after search, not memory.

## DO


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mantrakp04/g-spot](https://github.com/mantrakp04/g-spot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
