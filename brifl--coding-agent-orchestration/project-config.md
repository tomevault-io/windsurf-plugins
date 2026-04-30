---
trigger: always_on
description: This file defines the working contract for coding agents and humans collaborating in this repository.
---

# AGENTS.md — Execution Contract (Repo Baseline)

## Purpose

This file defines the working contract for coding agents and humans collaborating in this repository.

- This repo uses **project-local workflow files** stored under `.vibe/`.
- The agent must follow the **precedence order** and **stop conditions** below.
- This baseline is intended to be copied into target repos and may drift per project.

## Authoritative files and precedence

When instructions conflict, follow this order:

1. **User instructions in the current chat**
2. **This file: `AGENTS.md`**
3. **`.vibe/STATE.md`** (current truth: what we are doing now)
4. **`.vibe/PLAN.md`** (checkpoint backlog and acceptance criteria)
5. **`.vibe/HISTORY.md`** (non-authoritative rollups, context only)
6. Repository source code and tests
7. Anything else (including assumptions)

## Required behavior

### Read order at the start of a session

At the start of any new session (or when context is unclear), the agent must:
1. Read `AGENTS.md` (optional if already read this session)
2. Read `.vibe/STATE.md`
3. Read `.vibe/PLAN.md`
4. Optionally read `.vibe/HISTORY.md` if needed

### Work granularity

- Do **one coherent unit of work** at a time (typically one checkpoint step).
- Prefer **small, reviewable diffs**.
- When ambiguity affects correctness, ask **1–2 clarifying questions max** as issues in `.vibe/STATE.md`, then stop.

### Output discipline

- Keep outputs concise and structured.
- Prefer checklists, acceptance evidence, and exact commands over narrative.

### Quality bar (hard rule)

- The standard is **operator-trust quality**, not "the code changed" or "a placeholder output exists."
- The agent must actively understand the **user's real intent and the embodied/system behavior being built**, not just the literal wording of a task.
- Do not stop at the first technically passing condition if the live/user-visible behavior is still obviously below project intent.
- Quality gates must validate **meaningful behavior**, not just non-empty artifacts or green test surfaces.
- Before signoff, compare the result against what a reasonable operator would expect the system to actually do in the real world.
- If behavior is confusing, disappointing, or clearly below the intended bar, treat that as unfinished work even when tests pass.
- If the result is brittle, misleading, stub-like, or likely to fail the first real manual test, keep going or record a concrete blocker issue instead of declaring success.
- Prefer fewer honest `BLOCKED`/`IN_REVIEW` states over weak signoffs that create false confidence.

## Workflow model

### Checkpoint shape (required fields)

Each checkpoint should define:
- **Objective** (1 sentence)
- **Deliverables** (concrete files/modules/behaviors)
- **Acceptance** (verifiable conditions)
- **Demo commands** (exact local commands)
- **Evidence** (what to paste/link back into `.vibe/STATE.md`)

### Role loops

The agent should operate in one of these loops, as appropriate to `.vibe/STATE.md`.
This registry is canonical and must stay aligned with `.codex/skills/vibe-loop/scripts/agentctl.py` role mapping.

| Role key | Loop name | Intent | Typical transition |
| --- | --- | --- | --- |
| `design` | Stage Design | refine checkpoints in `.vibe/PLAN.md` | unclear/invalid status -> `design` |
| `implement` | Implementation | implement active checkpoint deliverables | `NOT_STARTED`/`IN_PROGRESS` -> `implement` |
| `review` | Review | verify acceptance and decide PASS/FAIL | `IN_REVIEW` -> `review` |
| `issues_triage` | Triage | resolve/clarify active issues with minimal scope | issues or blocked state -> `issues_triage` |
| `advance` | Advance | move pointer to next checkpoint (fallback/manual) | `DONE` + same-stage next checkpoint |
| `consolidation` | Consolidation | archive completed stage and sync docs/state | `DONE` + stage transition |
| `context_capture` | Context Capture | refresh `.vibe/CONTEXT.md` and clear context flags | post-consolidation or stale context |
| `improvements` | Process Improvement | improve orchestration system itself | workflow debt or explicit flag |
| `stop` | Stop | end loop when plan is exhausted | no remaining active checkpoints |

## Stop conditions (hard)

Stop and ask for input (as issue) if any of the following occurs:
- Missing required information to meet acceptance criteria
- Conflicting instructions between authoritative sources
- A decision point that changes scope, architecture, or dependencies materially
- The work would require secrets, credentials, destructive actions, or external side effects
- Tests/builds fail and the failure mode isn’t clearly attributable to the change

## Issue handling (lightweight)

- Track current issues in `.vibe/STATE.md` (active list only).
- Keep `.vibe/HISTORY.md` for resolved issues or postmortems.
- Don’t invent new “issue taxonomies” unless the repo explicitly needs it.

### Required active issue schema

Each active issue in `.vibe/STATE.md` should use this format:

```
- [ ] ISSUE-123: Short title
  - Impact: QUESTION|MINOR|MAJOR|BLOCKER
  - Status: OPEN|IN_PROGRESS|BLOCKED|RESOLVED
  - Owner: agent|human

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [brifl/coding-agent-orchestration](https://github.com/brifl/coding-agent-orchestration) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
