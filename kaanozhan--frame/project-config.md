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

**These ARE TASKS - add to .frame/tasks.json:**
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
2. Ask the user at an appropriate moment: "I identified these tasks from our conversation, should I add them to .frame/tasks.json?"
3. If the user approves, add to .frame/tasks.json

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

<!-- frame:managed:spec-section v=2 -->
## Spec-Driven Development (.frame/specs/)

Frame supports a structured `spec → plan → tasks → implement` workflow. When the user asks you to define, plan, or implement a feature, prefer this workflow over ad-hoc edits — it preserves intent and keeps `.frame/tasks.json` in sync.

### File layout

Each spec lives in its own folder:

```
.frame/specs/<slug>/
  spec.md       — what we're building
  plan.md       — how (architecture, files, footprint, sequencing)
  tasks.md      — flat bullet list, "- T01 · description"
  status.json   — phase + metadata
```

`<slug>` is kebab-case, derived from the spec title.

### Lifecycle phases

`draft` → `specified` → `planned` → `tasks_generated` → `implementing` → `done`

Frame auto-advances phase from filesystem state (file presence). The command templates below tell you exactly which `status.json` updates to make; Frame's watcher reconciles if anything is missed.

### Running spec commands — the self-serve protocol

The four spec commands are `spec.new`, `spec.plan`, `spec.tasks` and `spec.implement`. Whether the user types them as slash commands or asks conversationally ("plan the auth spec", "implement the tasks"), the flow is **never improvised from memory** — each command's current flow lives in a template file that Frame keeps staged in the project. Run one like this:

**1. Resolve the target spec.** An explicitly named spec always wins. Otherwise list the specs (`.frame/specs/*/status.json`) whose phase the command acts on — `spec.plan` → `specified`, `spec.tasks` → `planned`, `spec.implement` → `tasks_generated` or `implementing`. Exactly one candidate → take it silently; zero or several → present the candidates and ask. `spec.new` creates a new spec: derive the kebab-case slug from the title.

**2. Resolve the template.** Take the first that exists:

1. `.frame/templates/commands/<tool>/<command>.md` — project override
2. `.frame/runtime/commands/<tool>/<command>.md` — staged by Frame on project open

`<tool>` is the directory matching your CLI (Claude Code → `claude-code`). If neither file exists, say so and ask the user to open this project in Frame once so it stages the current templates — then stop. **Do not reconstruct the flow from this file, from memory, or from an older prompt.**

**3. Interpolate the placeholders.** Replace each `{placeholder}` token in the template:

| Placeholder | Value |
| --- | --- |
| `{project_path}` | absolute path of the project root |
| `{slug}` | the spec's slug |
| `{title}` | the spec's title (from `status.json`; for `spec.new`, the new title) |
| `{description}` | the user's description (`spec.new` only; empty otherwise) |
| `{report_template_path}` | `.frame/runtime/commands/<tool>/plan-report-template.html` |
| `{report_generator_path}` | `.frame/runtime/commands/<tool>/build-implement-report.mjs` |

**4. Follow the interpolated template exactly**, including every `status.json` update it prescribes. The template is the flow; this section only tells you how to find it.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kaanozhan/Frame](https://github.com/kaanozhan/Frame) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
