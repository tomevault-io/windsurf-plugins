---
trigger: always_on
description: This project is managed with **Frame**. AI assistants should follow the rules below to keep documentation up to date.
---

# TaskFlow - Frame Project

This project is managed with **Frame**. AI assistants should follow the rules below to keep documentation up to date.

> **Note:** This is the **sample project** that ships with Frame. It's a fictional codebase used to demonstrate Frame's workflow on realistic content. None of this code runs. When you're ready, open your own project to start real work.

---

## Task Management (tasks.json)

### Task Recognition Rules

**These ARE TASKS - add to tasks.json:**
- When the user requests a feature or change
- Decisions like "Let's do this", "Let's add this", "Improve this"
- Deferred work when we say "We'll do this later", "Let's leave it for now"
- Gaps or improvement opportunities discovered while coding
- Situations requiring bug fixes

**These are NOT TASKS:**
- Error messages and debugging sessions
- Questions, explanations, information exchange
- Temporary experiments and tests
- Work already completed and closed
- Instant fixes (like typo fixes)

### Task Creation Flow

1. Detect task patterns during conversation
2. Ask the user at an appropriate moment: "I identified these tasks from our conversation, should I add them to tasks.json?"
3. If the user approves, add to tasks.json

### Task Structure

```json
{
  "id": "unique-id",
  "title": "Short and clear title",
  "description": "Detailed explanation",
  "status": "pending | in_progress | completed",
  "priority": "high | medium | low",
  "context": "Where/how this task originated",
  "createdAt": "ISO date",
  "updatedAt": "ISO date",
  "completedAt": "ISO date | null"
}
```

### Task Status Updates

- When starting work on a task: `status: "in_progress"`
- When task is completed: `status: "completed"`, update `completedAt`
- After commit: Check and update the status of related tasks

---

## Spec-Driven Development (.frame/specs/)

Frame supports a structured `spec → plan → tasks → implement` workflow. When the user asks you to define, plan, or implement a feature, prefer this workflow over ad-hoc edits — it preserves intent and keeps `tasks.json` in sync.

### File layout

Each spec lives in its own folder:

```
.frame/specs/<slug>/
  spec.md       — what we're building (Problem, Goal, Constraints, Success Criteria, Out of Scope)
  plan.md       — how (Architecture, Files, Dependencies, Sequencing)
  tasks.md      — flat bullet list, "- T01 · description"
  status.json   — phase + metadata
```

`<slug>` is kebab-case, derived from the spec title.

### Lifecycle phases

`draft` → `specified` → `planned` → `tasks_generated` → `implementing` → `done`

Frame auto-advances phase from filesystem state (file presence). After writing each artifact, update `status.json` so `phase`, `updated_at`, and `last_phase_at` reflect reality — Frame's watcher will reconcile if you forget.

### Slash commands

When the user types a Frame slash command, write **exactly one file** and then update `status.json`:

- `/spec.new <description>` → write `spec.md` (sections: Problem, Goal, Constraints, Success Criteria, Out of Scope). Phase → `specified`.
- `/spec.plan` → read `spec.md`, write `plan.md` (sections: Architecture, Files, Dependencies, Sequencing). Phase → `planned`.
- `/spec.tasks` → read `spec.md` + `plan.md`, write `tasks.md` as a flat `- T01 · ...` bullet list (5–12 tasks, imperative voice). Phase → `tasks_generated`.

After `/spec.tasks`, **do not** also write entries to `tasks.json` — Frame's watcher imports them automatically with `source: "spec:<slug>:T<n>"` markers.

### tasks.json linkage

Spec-generated tasks carry a `source` field. Treat them like any other task — start them, complete them, update status. User-set status is preserved across spec re-imports; only title/description sync from `tasks.md`.

### When to suggest a spec (steer the conversation)

Spec-driven is Frame's core way of working, so when a user describes meaningful new work **mid-conversation**, gently steer them toward a spec instead of silently diving into code. Suggest a spec only for **significant work** — don't make this a reflex on every message.

**Suggest a spec for:**
- A new **feature** or capability ("users should be able to …", "add a … system")
- A change that will touch **multiple files / modules** or affect architecture
- Anything that clearly benefits from a **plan and ordered tasks** before coding
- Work the user describes vaguely/largely that would benefit from being scoped first

**Do NOT suggest a spec for:**
- Typos, one-line fixes, small tweaks, renames → just do it
- Small, discrete tracked work → that's a task (`tasks.json`)
- Questions, debugging, explanations, experiments
- Anything the user explicitly says to "just do" / "do directly"

Rough ladder: *trivial → just do it · small but worth tracking → task · sizable feature or multi-file change → spec.*


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kaanozhan/Frame](https://github.com/kaanozhan/Frame) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
