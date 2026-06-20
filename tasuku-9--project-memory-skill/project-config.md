---
trigger: always_on
description: >
---


# Project Memory Skill

Use this skill when the user wants to preserve, recover, migrate, or audit long-running project or research context across chat loss, model changes, tool handoffs, or multi-session work.

The skill separates **current truth**, **plans**, **decisions**, **evidence**, **hypotheses**, **human summaries**, and **recovery checkpoints** so that no single chat transcript becomes the source of truth.

## When to use

Use this skill when the user asks to:

- start a new project and set up structured memory from scratch
- introduce structured memory into an existing project with accumulated context
- resume work after a lost or interrupted chat
- create a durable project memory or research log
- migrate context from one model, agent, workspace, or repository to another
- update project docs after new work, decisions, research, experiments, or failures
- produce a handoff brief for a human or another model
- distinguish confirmed facts from hypotheses, plans, and unresolved questions
- audit whether a project has enough continuity documentation
- manage a research paper, thesis, publication, or traceability-heavy project: track literature, connect findings to hypotheses, log figures and tables with their data sources

Do not use this skill as a replacement for domain expertise, citations, or source verification. It is a continuity and documentation-routing skill.

## Operating assumption

The human usually does not write memory files directly.
The AI agent is responsible for updating the project memory during the chat or coding session.

Do not rely on tool-internal or session-local memory as the canonical source of truth.
Treat repository files as the durable, shared memory layer across Codex, Claude Code, and future tools.

## Core principle

Capture broadly. Promote narrowly.

Do not let hypotheses, plans, or recovery notes silently become truth.

## Routed memory rule

Project memory is not a full-file synchronization system.

Do not read or update every project-memory file on every run.
Do not treat the file list or read order as a checklist.

Each piece of information should go to its canonical home.
Update only the files whose canonical responsibility changed.

Most work sessions should update only 1-3 files.
Larger updates are appropriate only during major transitions, such as phase changes, major hypothesis confirmation or rejection, architecture or approach changes, release checkpoints, or large migrations.

Before editing memory files, produce a short update plan stating:

- which files will be updated
- why each file needs an update
- which relevant files will not be touched

For a small, obvious one-file update, a one-sentence plan is enough.
If you notice you are about to edit memory files without an update plan, stop and produce the plan first.

When one piece of information seems to belong in multiple files, choose one canonical home according to `DOCS_GUIDE.md` and `CONTEXT_MANIFEST.md`.
Other files may reference the canonical entry, but should not duplicate the full details.

## Read scope rule

Start with `CONTEXT_MANIFEST.md` when present.

Do not preemptively read the full memory set.
Use the read order as a priority order, not as a checklist.

Before reading additional memory files, identify which files are relevant to the current task.
For non-trivial work, briefly state which memory files will be read and which relevant files will not be read.

## Conversation capture rule

During a work session, do not wait for the human to explicitly ask for logging.

Do not write to canonical memory files at the first appearance of an idea.
Treat new hypotheses, decisions, observations, blockers, risks, and next actions as capture candidates until they stabilize.
Keep capture candidates silently during the current session.
Do not announce every candidate or repeatedly say that it is being held for later.

Write capture candidates to memory when the topic changes, the discussion reaches a conclusion, the session is ending or may be interrupted, the human explicitly asks to record something, or a decision, result, blocker, or next action becomes clear enough to preserve.
After writing, briefly report what was recorded and where.

Capture only material that changes project memory.
Do not summarize routine conversation, temporary phrasing, or early ideas that are still being refined.

Use `RECOVERY_NOTES.md` only as a short resume pointer.
Do not make it the source of truth.

## Profile and customization rule

project-memory is a skeleton, not a fixed operating system.
Customize the profile, capture trigger strength, review cadence, archive policy, and tool-facing instructions to fit the project.

When initializing an empty workspace or adopting an existing project, recommend the smallest sufficient profile before writing memory files:

- `light`: small personal project, short-lived work, or minimal continuity
- `standard`: normal coding, writing, or product work with decisions and next actions
- `research`: experiments, evidence, hypotheses, repeated investigation, or debugging loops
- `academic`: literature, figures, tables, thesis, paper, provenance, or publication-grade traceability workflow

Recommend a capture trigger strength alongside the profile:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tasuku-9/project-memory-skill](https://github.com/tasuku-9/project-memory-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
