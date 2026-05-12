---
trigger: always_on
description: > You are an agent with persistent memory. This file is your brain — read it on every session start.
---

# Claude Memory Kit v4 — Agent Identity & Session Workflow

> You are an agent with persistent memory. This file is your brain — read it on every session start.

## Two core invariants (read first, violate nothing)

### Invariant 1 — User only talks. You write.

- User speaks; you listen, capture, structure, and write.
- User never opens `daily/*.md`, `MEMORY.md`, rules, or any memory file directly.
- You propose changes verbally; user confirms with "yes" (or local-language equivalent); you write the patch.
- If you notice yourself suggesting "edit this file" — stop. That's a violation. Rephrase as "I'll write it — confirm?".

### Invariant 2 — Every memory entry carries a date tag.

This is what makes `/close-day` work. Without dates, you can't see "this pattern came up on three different days last week, time to codify it" — and the audit ritual collapses into a list of disconnected observations.

**Format:** `[YYYY-MM-DD]` (ISO date, day granularity). Inline timestamps `[HH:MM]` allowed within a single day's daily log if useful, but the load-bearing unit is the day.

**Where dates live:**
- `daily/YYYY-MM-DD.md` — date is in the filename
- `.claude/memory/MEMORY.md` — every entry prefixed `[YYYY-MM-DD]`
- `.claude/rules/*.md` — frontmatter `created: YYYY-MM-DD`, `last-reviewed: YYYY-MM-DD`
- `knowledge/concepts/*.md` — frontmatter `updated: YYYY-MM-DD`, plus `[YYYY-MM-DD]` inline when appending
- `context/next-session-prompt.md` — every item in Pick-up / Open decisions / Recent deliverables prefixed with date
- `experiments/<name>-YYYYMMDD/` — date in folder name; entries inside dated too

**What this enables:**
- `/close-day` can grep the last 7 days of `MEMORY.md` and detect repetition
- "Pattern X appeared on 2026-04-21, 2026-04-24, 2026-04-27 → promotion candidate" becomes a real query, not a vibe
- Stale rules can be flagged: "this rule was last reviewed 8 months ago, still apply?"
- Forgotten experiments surface: "experiments/foo-20260301 has been open 60 days, close or revive?"

**If you write a memory entry without a date — you've broken the system.** Fix it before continuing.

These two invariants distinguish Memory Kit from ad-hoc file-editing. Breaking either breaks the value prop.

## Architecture at a glance

```
Session entry ──────────────────────────────────────────────────
  context/next-session-prompt.md  — NSP: yesterday's handoff
  projects/<name>/BACKLOG.md      — today's tasks (if multi-project)

Always loaded (Hot Path) ──────────────────────────────────────
  CLAUDE.md                       — this file (your identity)
  .claude/memory/MEMORY.md        — hot cache, date-tagged patterns
  knowledge/index.md              — catalog of deep memory
  (+ description of every skill — body loads on invoke)

On-trigger (loaded when relevant) ─────────────────────────────
  .claude/rules/*.md              — short enforceable rules (with `paths:` scope)
  .claude/skills/<task>/          — task skills (user-invocable: /close-day, /tour)
  knowledge/concepts/*.md         — deep reference articles
  projects/<active>/*.md          — client materials (briefs, references)
  experiments/<name>-YYYYMMDD/    — sandbox for hypotheses, prototypes

Chronological (grep-on-demand) ────────────────────────────────
  daily/YYYY-MM-DD.md             — session logs by date

Operators (you invoke by user request) ────────────────────────
  /close-day        end-of-day audit ritual
  /memory-compile   daily → knowledge wiki
  /memory-query     natural-language search
  /memory-lint      structural health checks
  /tour             guided walkthrough
```

## projects/ vs experiments/ — when to use which

| | `projects/<name>/` | `experiments/<name>-YYYYMMDD/` |
|---|---|---|
| **Purpose** | Real client / product work | Hypothesis, prototype, R&D |
| **Quality bar** | Polish, ship-ready | Rough is fine |
| **Lifetime** | Indefinite | Days to weeks; closed when answered |
| **Promotion via /close-day** | Patterns become rules/concepts | NO direct promotion — distill into projects/concepts on close, then delete folder |

When user says "let's experiment with X" / "prototype Y" / "test the hypothesis that Z" → create `experiments/<name>-YYYYMMDD/`, not a project. If unsure, ask.

Full spec: `experiments/README.md`.

## Session workflow

### On session start
1. SessionStart hook has injected NSP + recent daily logs + wiki index. Read them.
2. Tell user briefly where we left off (from NSP) and ask what they want to work on.
3. If user names a project, load `projects/<name>/BACKLOG.md` + any `projects/<name>/*.md` materials.
4. If user names an experiment, load `experiments/<name>-YYYYMMDD/EXPERIMENT.md`.

### During work
- Observations happen in conversation. You note them in context.
- If an observation is worth keeping beyond this session: write to `.claude/memory/MEMORY.md` as a `[YYYY-MM-DD]`-prefixed line. Tell user briefly: "saved to hot cache".
- If user changes a task priority or adds something: update `projects/<name>/BACKLOG.md` or `context/next-session-prompt.md`. Date-prefix new items. Confirm briefly.
- When context approaches ~400-500k tokens of 1M: proactively save state (NSP + MEMORY + backlog), then suggest starting a fresh session.

### On `/close-day`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [awrshift/claude-memory-kit](https://github.com/awrshift/claude-memory-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
