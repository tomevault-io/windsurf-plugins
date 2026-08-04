---
trigger: always_on
description: This file provides guidance to Claude Code when working in this repository.
---

This file provides guidance to Claude Code when working in this repository.

# Project purpose

Build a real Delphi Win64 debugger for VS Code using DAP + Windows Debug API.

Primary goal is progress on the debugger itself.
The sample target app exists only for testing.
It may be extended whenever needed to validate new debugger features.

# Operating mode

Use minimum tokens.

Be concise.
No filler.
No praise.
No motivational tone.
No repeating the request.
No long plans unless asked.
No unnecessary explanations.

If code requested: output code first.
If uncertain: say uncertain.
If idea is bad: say so directly.

Prefer practical solutions over academic ones.

# Critical token rules

Assume context window and usage budget are scarce.

Avoid re-reading many files unless necessary.
Avoid broad repo scans.
Inspect only files relevant to the current task.
Do not restate known project context.

Prefer direct edits over discussion.

When multiple approaches exist:
choose the smallest production-relevant step.
Do not implement shortcuts that only work for Debugme.
Every solution must be compatible with real Delphi Win64 applications unless explicitly marked as prototype-only.
Prefer incremental progress, but never hardcode assumptions from the test project.

# Prototype vs production

Debugme is only a test target.

Never design features around Debugme-specific assumptions.

Allowed:
- using Debugme to reproduce and validate behavior
- extending Debugme to cover newly implemented debugger features
- implementing narrow incremental pieces

Not allowed:
- hardcoded source paths
- hardcoded module names
- hardcoded symbol names
- assumptions about one unit, one source file, one thread, one stack frame, or one module
- solutions that work only because Debugme is trivial

If a temporary shortcut is unavoidable:
- mark it clearly with TODO PROTOTYPE
- document what must change for real projects
- update TASK_RESUME.md with the limitation

# Session continuity rules

Maintain these files:

- PROJECT_STATE.md   (high-level permanent state)
- TASK_RESUME.md     (exact current task state)

Update TASK_RESUME.md continuously enough that work can resume after an interruption in the middle of a long step.

Do not wait for a full step to finish.

Update TASK_RESUME.md whenever restart cost is starting to rise, especially after:

- a non-trivial discovery
- a code edit
- a test/build result
- a local change of hypothesis
- a switch of file, symbol, or investigation path
- any sign that token budget may run out before the current step is complete

If usage seems near limit:
stop coding and update TASK_RESUME.md first.

TASK_RESUME.md must contain:

- current task
- current substep
- current files / symbols in focus
- last completed action
- next action if interrupted right now
- files involved
- what works
- what is failing
- last test result
- exact next step
- traps / hypotheses

When a long step is still in progress, TASK_RESUME.md must describe the current cursor inside that step, not just the high-level milestone.

The cursor should be brief but specific enough that a new session can continue without re-reading broad context.

PROJECT_STATE.md must contain:

- architecture status
- implemented features
- open milestones
- important technical discoveries
- stable build/run commands

PROJECT_STATE.md must not duplicate transient task state that belongs in TASK_RESUME.md.

# Living specifications

The following documents at the repository root are living specifications.
They describe state of knowledge about the project's formats and
architecture and are maintained continuously alongside the code:

- `RSM_FORMAT_NOTES.md` — overall structure of the Delphi `.rsm` file.
- `RSM_RECORD_TYPES.md` — catalog of tags / record kinds with confirmed /
  inferred / conjectured status.
- `RSM_FIELD_OFFSETS.md` — byte-level layout of each record.
- `DAP_DEBUGGER_ARCHITECTURE.md` — modules, threading model, breakpoint /
  evaluate / setVariable flows, capability list.
- `KNOWN_UNKNOWNS.md` — open questions that block or condition the work.

Rules:

- Before investigating anything related to `.rsm`, the adapter
  architecture, or open questions: read the relevant document first. Do
  not re-derive what is already written.
- When you discover or confirm a fact: update the relevant document in
  the same change set as the code or experiment that produced the fact.
- When an entry in `KNOWN_UNKNOWNS.md` is resolved: move the answer into
  whichever document now owns it (`RSM_*`, `DAP_DEBUGGER_*`,
  `PROJECT_STATE.md`) and remove the entry from `KNOWN_UNKNOWNS.md`. Do
  not leave resolved questions there for historical reference.
- If a document disagrees with the code: the code wins. Correct the
  document, do not bend the code to match the document.

# Resume behavior

When starting a new session:

1. Read PROJECT_STATE.md
2. Read TASK_RESUME.md
3. Read the living specifications relevant to the next step:
   - work on symbols / locals / globals / types → the `RSM_*` documents
   - work on the adapter, debug loop, DAP requests, stepping → `DAP_DEBUGGER_ARCHITECTURE.md`
   - in every session, regardless of focus → `KNOWN_UNKNOWNS.md`
4. Inspect only referenced files first
5. Resume exactly from next step


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [csm101/delphi-visual-studio-code-debugger](https://github.com/csm101/delphi-visual-studio-code-debugger) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->
