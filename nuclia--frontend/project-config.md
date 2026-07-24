---
trigger: always_on
description: Nuclia frontend Nx monorepo. See `AGENTS.md` for workspace structure and `README.md` for project setup.
---

# GitHub Copilot Instructions

Nuclia frontend Nx monorepo. See `AGENTS.md` for workspace structure and `README.md` for project setup.

---

## Core Workflow Rules

### 1. Validate Before Executing

**Do not start implementing until the task is clear.** Before writing any code:

1. **Restate the task** in one sentence to confirm understanding.
2. **Identify ambiguities** — which project? which component? what's the expected behaviour?
3. **Ask open questions** if anything is unclear. Prefer open-ended questions over yes/no when you need context. A 50-token question is always cheaper than a 5000-token wrong implementation.
4. **State your plan** briefly before starting (2-3 bullet points, not a wall of text).

When to ask vs. when to infer:

- **Ask** when: the target project is ambiguous, the scope could change which files are touched, or a wrong assumption would require significant rework.
- **Infer** when: the project can be determined from file paths or previous messages, the task is purely additive, or the user has already answered a similar question in this session.

### 2. Anti-Cycling Protocol

Token waste from spinning is the biggest efficiency killer. Follow these rules strictly:

- **3-attempt maximum.** If the same approach fails 3 times (build error, test failure, lint error), STOP. Explain what you tried, what failed, and ask the user for guidance.
- **Pivot after 2.** If 2 attempts at the same approach fail, try a fundamentally different approach on the 3rd attempt — don't iterate on a broken idea.
- **No speculative exploration.** Don't read 20 files "just in case." Read the files you need, act, verify. If you need more context, read more files — but always with a specific question in mind.
- **Detect your own loops.** If you notice you're doing the same grep/read/edit cycle without making progress, stop and tell the user what's blocking you.
- **Scope-check long tasks.** If a task is taking more than ~10 tool calls without visible progress, pause and reassess: is the approach wrong? Is the task bigger than expected? Tell the user.

### 3. Task Sizing

Before diving in, mentally classify the task:

| Size   | Description                                         | Approach                                      |
| ------ | --------------------------------------------------- | --------------------------------------------- |
| **XS** | Typo fix, rename, single-line change                | Do it directly, no skill loading needed       |
| **S**  | Modify one component/service, add a translation key | Load relevant skill if pattern is non-obvious |
| **M**  | New component, new service, new route               | Load skill + read project AGENTS.md           |
| **L**  | Multi-file feature, cross-project change            | Consider using sub-agents via orchestrator    |
| **XL** | New library, major refactor, multi-domain feature   | Use orchestrator with full agent delegation   |

**Don't bring a cannon to a knife fight.** An XS task should not trigger skill loading, agent delegation, or orchestrator coordination.

---

## Session Knowledge Protocol

Long sessions produce valuable knowledge. Capture it so future sessions don't start cold.

### What to capture

- **Non-obvious constraints:** "Component X breaks if you remove NgModule Y because of Z"
- **Workarounds:** "Use `setTimeout` wrapper for signal X due to Angular change detection timing"
- **Hidden dependencies:** "Service A must be initialized before Service B — no error, just silent failure"
- **Architecture decisions:** "Team chose approach X over Y because of Z"
- **DO NOT capture:** standard patterns (already in skills), obvious things, session-specific temporary state

### Where to write it

| Knowledge type                      | Write to                 | Section              |
| ----------------------------------- | ------------------------ | -------------------- |
| Project-specific gotcha (1-2 lines) | Project's `AGENTS.md`    | `## Gotchas` section |
| Deeper explanation (>3 lines)       | Project's `KNOWLEDGE.md` | Relevant heading     |
| Cross-project pattern               | Root `AGENTS.md`         | `## Key Conventions` |

### When to write it

- **At natural breakpoints** in long sessions (feature completed, bug fixed, refactor done)
- **When explicitly asked** ("save what you learned", "update knowledge")
- **When you discover a gotcha** that would have saved time if you'd known it earlier
- **End of long sessions** — offer to update AGENTS.md with any discoveries

### KNOWLEDGE.md format

Each project may have a `KNOWLEDGE.md` alongside its `AGENTS.md`. Link it from AGENTS.md:

```markdown
## Deep Knowledge

For non-obvious patterns, workarounds, and architectural decisions, see [KNOWLEDGE.md](./KNOWLEDGE.md).
```

KNOWLEDGE.md entries should be:

- **Dated** (month/year is fine)
- **Titled** with the problem or topic
- **Actionable** — what to do or avoid, not just "here's what happened"

---

## Cross-Session Memory

A session store records every session's files, tools, and conversations. **Use it to avoid re-discovering what was already learned.**

Before starting work on a component or feature area, check if prior sessions touched the same files:

```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nuclia/frontend](https://github.com/nuclia/frontend) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
