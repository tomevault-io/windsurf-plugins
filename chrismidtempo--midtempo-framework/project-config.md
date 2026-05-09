---
trigger: always_on
description: Last Updated: 22/04/2026
---

# Agent Rules

Last Updated: 22/04/2026

**Agent's Core Principle:** Do the simplest thing that works.

---

## Table of Contents

- [Pre-Action Gate](#0-pre-action-gate)
  - [Step 1: Route to Appropriate Skill](#step-2-route-to-appropriate-skill)
  - [Step 2: Enforce Skill-Based Code Changes](#step-3-enforce-skill-based-code-changes)
- [Quick Reference](#1-quick-reference)
- [Iron Laws](#2-iron-laws)
- [Workflow](#3-workflow)
- [Skill Router](#4-skill-router)
  - [Skills](#41-skills)
  - [Task Skills](#42-task-skills)
  - [Review Skills](#43-review-skills)
  - [Routing Rules](#44-routing-rules)
- [Writing Style](#5-writing-style)
- [Tooling](#6-tooling)
  - [Core Tooling](#61-core-tooling)
  - [Additional Tooling](#62-additional-tooling)
  - [Tooling Rules](#63-tooling-rules)
- [Bypass Protection](#6-bypass-protection)
- [Stop Conditions — ASK HUMAN](#7-stop-conditions--ask-human)

---

## 0. Pre-Action Gate

### Step 1: Route to Appropriate Skill

**REQUIRED ACTION:**
```
SCAN user request for trigger words (see "§4. Skill Router")
SCAN SCOPE: includes the content of any referenced or attached documents, not just the user's typed message.

IF referenced document contains a **Skill:** field
  → Treat as a direct skill routing instruction
  → READ this ENTIRE file (CLAUDE.md) first
  → THEN read the skill file specified in the **Skill:** field
  → FOLLOW skill instructions exactly

IF TRIGGER WORDS FOUND:
  → READ this ENTIRE file (CLAUDE.md) first
  → THEN read the matching skill file
  → FOLLOW skill instructions exactly

IF NO TRIGGER WORDS FOUND:
  → Proceed to "Step 3: Enforce Skill-Based Code Changes"
```

### Step 2: Enforce Skill-Based Code Changes

**MANDATORY:** Before making any code changes

**REQUIRED ACTION:**
```
IF request involves changing production code, tests, or configuration:
  AND no `midtempo-framework/*.md` skill is active:
    → STOP immediately
    → DO NOT make code changes
    → ASK user which skill to run
    → EXPLAIN: "Code changes require an midtempo-framework skill (build, bug, refactor, refine)"
    → WAIT for user to specify skill or approve ad-hoc change

IF simple non-code tasks (documentation only, research, file reading):
  → VALID: Proceed with task
```

**MANDATORY RULE:** NEVER MAKE CODE CHANGES WITHOUT AGENTIC-FRAMEWORK SKILL

---

## 1. Quick Reference

- **Test Driven Development always** — never production code without a failing test.
- **Files ≤ 500 lines, functions ≤ 75 lines**
- **UK English** throughout
- **No `console.log`** — only `.warn`/`.error` 
- **No disabling lint**  without human approval
- **Coverage thresholds:** lines 90%, functions 90%, branches 70%
- **Test output:** `/planning/last-test-ran.log`
- **Commit messages:** No Attribution or Co-Authored-By tags

---

## 2. Iron Laws

These rules cannot be waived. Skill files enforce them; violations trigger automatic recovery.

| Law        | Rule                                     | Skill                      |
| ---------- | ---------------------------------------- | -------------------------- |
| TEST DRIVEN DEVELOPMENT (TDD)        | No production code without failing test  | `/midtempo-framework/rules/tdd.md`     |
| BUGS       | Trace to root cause, never fix symptoms  | `/midtempo-framework/bugs.md`          |
| REFACTOR   | Only on green, behaviour unchanged       | `/midtempo-framework/refactor.md`      |
| TESTING    | Test real behaviour, not mocks           | `/midtempo-framework/rules/testing.md` |
| COMPLETION | No placeholders, implement fully or omit | —                          |

Note: All Dates in DD/MM/YYYY format.

---

## 3. Workflow

All feature work follows this sequence. No phase may be skipped. Human approval required at each step.

```
BUILD → DESIGN DOC → PLAN DOC → TEST DOC → RED → GREEN → REFACTOR → UPDATE DOCS
```

| Phase         | Skill                      | Output                        |
| ------------- | -------------------------- | ----------------------------- |
| Build    | `/midtempo-framework/build.md` | `planning/[feature]-decisions.md` |
| Design    | `/midtempo-framework/write-design.md` | `planning/[feature]-design.md` |
| Plan          | `/midtempo-framework/write-plan.md`    | `planning/[feature]-plan.md`   |
| Test Manifest | `/midtempo-framework/write-tests.md` | `planning/[feature]-tests.md`  |
| Red → Green → Refactor → Docs    | `/midtempo-framework/deliver.md`       | Working code, tests, docs     |

---

## 4. Skill Router

**Before responding to any request, identify the matching skill and read it FIRST.**

Skills do not stack. If multiple triggers match, clarify which skill applies.

### 4.1 Skills

These drive the delivery pipeline. Each produces artefacts for the next phase.

| Trigger Words                      | Skill                      | Purpose                         | Output                        |
| ---------------------------------- | -------------------------- | ------------------------------- | ----------------------------- |
| setup     | `/midtempo-framework/setup.md` | Setup the midtempo-framework for the repo      | `/midtempo-framework/instructions/*.md` files |
| idea, brainstorm, build, explore, create     | `/midtempo-framework/build.md` | Transform idea into design      | `planning/[feature]-design.md` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ChrisMidtempo/midtempo-framework](https://github.com/ChrisMidtempo/midtempo-framework) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-26 -->
