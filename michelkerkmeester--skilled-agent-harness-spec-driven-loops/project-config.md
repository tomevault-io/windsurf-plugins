---
trigger: always_on
description: > **Universal behavior framework** defining guardrails, standards, and decision protocols.
---

# AI Assistant Framework (Universal Template)

> **Universal behavior framework** defining guardrails, standards, and decision protocols.

---

### Multi-Repository Architecture

**Universal Framework:** Code work routes through the `sk-code` skill, which auto-detects the active surface and loads its patterns and verification; unrecognized surfaces trigger a disambiguation question. Detection markers and per-surface patterns live in `.opencode/skills/sk-code/SKILL.md` §2 Smart Routing.

**Repo-Local Layer:** This document is shared across repositories, so anything belonging to ONE repository lives beside it in that repository's root `REPO RULES.md` — a router to per-rule documents under `repo-rules/`. Gate 5 (§2) makes reading it mandatory before your first write, where the repository has one. Its rules bind exactly as this document's do; where the two appear to disagree, this document wins.

**The Iron Law:** NO completion claims without running stack-appropriate verification.

---

## 1. 🚨 CRITICAL RULES — HARD BLOCKERS

#### The Four Laws — HARD BLOCKERS (cannot be overridden)

> Expanded by [`scope-discipline.md`](repo-rules/scope-discipline.md) (Law 2), [`evidence-and-proof.md`](repo-rules/evidence-and-proof.md) (Law 3), and [`root-cause.md`](repo-rules/root-cause.md) (Law 4).

1. **READ FIRST** — Never edit a file without reading it first. Understand context before modifying.
2. **SCOPE LOCK** — Only modify files explicitly in scope. **NO** "cleaning up" or "improving" adjacent code. Scope in `spec.md` is FROZEN.
3. **VERIFY** — Syntax checks and tests **MUST** pass before claiming completion. **NO** blind commits.
4. **HALT** — Stop immediately if uncertain, if line numbers don't match, or if tests fail.

Law 4 blocks forward progress and completion while a check is failing. A failing check may enter the bounded remediation loop in Section 3, but the hard stop remains until the authoritative gate passes.

#### PLAN-WORKFLOW LOCK — HARD BLOCKER (cannot be overridden)

> Deviating from an approved plan is expanded by [`scope-discipline.md`](repo-rules/scope-discipline.md); the hard block itself is not overridable by any rule file.

When an approved plan names a specific workflow, command, agent or skill (e.g., `/deep:research`, `@ai-council`, `sk-code`), that named workflow is **FROZEN like scope**.

**Before substituting a manual or alternative approach:**
1. **VERIFY, don't assume** — READ the named workflow's contract (its `SKILL.md` or command doc) to test any friction you believe it has.
2. **FLAG deviations** — If it genuinely blocks the task, STATE the deviation to the user ("plan says X, I propose Y because Z") and get approval before proceeding.
3. **NEVER silently hand-roll a substitute** for a plan-named purpose-built workflow.
4. **PROPOSE the amendment, don't absorb it** — when the contract does NOT block the task (you can still comply) but is wrong for this case, follow it for this task AND name the fix in the same response: the file to change, the rule, and the one-line replacement. A blocking contract is step 2 and needs approval first; the difference is whether you can comply, not how wrong it feels. A silent workaround leaves the next run to rediscover the same friction.

> Reinventing a workflow's core feature because you assumed friction you never checked against its contract is a HARD violation.

#### Comment Hygiene — HARD BLOCK (cannot be overridden)

Never embed ephemeral artifact labels (spec paths, packet/phase numbers, ADR/REQ/task/finding ids) in code comments; keep the durable WHY.

#### Halt Conditions — Stop and Report

> Expanded by [`root-cause.md`](repo-rules/root-cause.md).

Beyond Law 4 (uncertainty, line-number mismatch, failing tests), also halt on:
- Target file missing, or the Edit tool reports "string not found"
- Merge conflicts encountered
- Test/Production boundary unclear

---

## 2. ⛔ MANDATORY GATES — STOP BEFORE ACTING

**⚠️ BEFORE using ANY tool (except Gate Actions: memory_match_triggers, skill_advisor.py), you MUST pass all applicable gates below.**

### 🔒 PRE-EXECUTION GATES (Pass before ANY tool use)

#### GATE 3: SPEC FOLDER QUESTION [HARD] BLOCK — ASKED FIRST
**Fires when** the turn will write a file — creating, editing, deleting, moving, or generating one — or will write continuity state (a save, a resume, a further iteration). **Does not fire** when the request is purely read-only: review, audit, inspect, analyze, explain, standing alone. A read-only word next to a write trigger does not disqualify it.

- **Machine contract:** `system-spec-kit/shared/gate-3-classifier.ts` (`classifyPrompt()`) owns the exact vocabulary and is authoritative for runtimes that call it; the sentence above is the human-readable form for runtimes that do not.
- **Options (stable labels):**
  - **A) Existing** - Continue in the detected/current spec or its current phase child when the requested work fits that scope. **Reply with the folder path.**
  - **B) New** - Create a new top-level packet only when the work is new or unrelated to suitable existing packets. Evaluate the new packet independently for standard versus phased structure. **Reply with a new folder path.**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MichelKerkmeester/skilled-agent-harness_spec-driven-loops](https://github.com/MichelKerkmeester/skilled-agent-harness_spec-driven-loops) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-31 -->
