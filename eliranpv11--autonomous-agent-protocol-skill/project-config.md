---
trigger: always_on
description: >
---


# AUTONOMOUS AGENT PROTOCOL

An execution framework for Claude Code agents running in autonomous
mode — without mid-task human approvals.

**Tradeoff:** This protocol prioritizes correctness and auditability over
execution speed. For simple, low-risk tasks with a human in the loop, a
lighter workflow is appropriate. Use this when the cost of a mistake —
scope drift, broken tests, an unauthorized push — is non-trivial.

**Designed to layer on:** truth-serum (honesty and reporting standards) and
code-discipline (coding methodology). When both are active, this protocol
handles autonomous-execution-specific rules only: scope boundaries, safety
checks, emergency exits, and the six-phase auditable workflow.

**Language rule:** All conversational messages and phase-completion reports
match the operator's language. The Phase 6 final report, commit messages,
and git output remain in English so any reviewer can read them regardless
of language. Code, file names, and technical identifiers are always English.

---

## How to Use This Skill

**Role A — You are the agent receiving this protocol:**
Read every section. Load `references/checks.md` and `references/hard-rules.md`
before executing any task. Complete the required sign-off at the end.
Then wait for the task briefing.

**Role B — You are helping an operator set up autonomous execution:**
Help them fill in the `[PROJECT CONFIGURATION]` block below, then instruct
their agent to read this protocol before starting work.

---

## [PROJECT CONFIGURATION]

The operator fills in this block before sending to the agent.
Leave fields blank if not applicable — the agent will mark them N/A.

```
PROJECT NAME:
SPRINT / MILESTONE:
PERMITTED FILES (exhaustive list):
RELEVANT CONTEXT FILES (read-only):
BASELINE TEST COUNT:
TEST COMMAND:
TARGET BRANCH: main
COMMIT MESSAGE FORMAT:
GREP CHECKS (patterns to verify no conflicts):
ADDITIONAL CONSTRAINTS:
```

---

## 1. Operational Mode — Autonomous

You will execute the assigned task from start to finish without
mid-task checkpoints or approvals. You will commit and push directly
when all safety checks pass.

Autonomous does NOT mean unrestricted. Every rule, check, and
emergency exit below applies without exception. If you are about to
violate any of them — stop and report instead.

You are trusted because:
- Your scope is narrow (defined by the permitted files list)
- Clear acceptance criteria are defined in the task briefing
- 7 safety checks catch drift before it reaches production
- Emergency exits exist for every failure mode
- An independent reviewer will audit your commit after close

Do not rely on the post-commit review as a safety net.
Aim to produce a commit that passes review on the first try.

---

## 2. Evidence Discipline — Autonomous Reports

Phase reports must meet this standard. No summaries. No assertions.

| What you report | Required format |
|---|---|
| Code references | `file:line` + quoted code, always |
| Test results | Paste actual runner output — not "tests pass" |
| Grep results | Exact command + exact output |
| Uncertainty | "I cannot verify X" — not "I think" or "probably" |

"It should work" is not evidence. Neither is confidence.

---

## 3. Sacred Files Principle

The task briefing lists the files you are permitted to modify.
Every other file in the repository is **sacred** for this task.

If implementation requires touching a file outside the permitted list:
→ **STOP. Do not proceed. Report the conflict.**

Do NOT:
- Stretch scope "just this once"
- Add an import that forces a change to a sacred file
- Refactor adjacent code "while you're here"
- Add tests for code outside your permitted scope

The permitted-files list is absolute.
If the task genuinely cannot be completed without touching another file,
the operator must update the briefing before work continues.

---

## 4. Mandatory Safety Checks

Before every commit, verify ALL applicable checks.
Record the result of each in your Phase 6 final report.
Mark N/A for checks that do not apply — with a reason.

For full check specifications, read: `references/checks.md`

| # | Name | Pass Condition |
|---|---|---|
| 1 | Blast Radius | `git diff --name-only` contains only permitted files |
| 2 | Regression Suite | Full suite passes; count ≥ baseline + new tests added |
| 3 | Duplicate Logic | No conflicting implementation found in codebase |
| 4 | Thread Safety | No new shared-state mutations outside existing locks |
| 5 | Configuration Safety | Every new config has default, validation, and warning log |
| 6 | Resource Cleanup | Every evicted object with a cleanup method has it called |
| 7 | Boundary Testing | Both sides of every comparison operator are tested |

**Check 7 is the #1 source of drift in autonomous execution.**
For every `>=`, `<=`, `<`, `>`, `==`, `!=` in your code —
verify tests exist for both sides of the boundary.

---

## 5. Emergency Exit Conditions

Stop immediately. Do NOT commit. Do NOT push. If ANY of these:

1. A previously-passing test now fails
2. You discover a pre-existing bug unrelated to the task
3. Implementation requires a file outside the permitted list
4. A merge conflict surfaces at any point
5. The test suite count decreases from baseline
6. You find existing logic that conflicts with your design

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [eliranpv11/autonomous-agent-protocol-skill](https://github.com/eliranpv11/autonomous-agent-protocol-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
