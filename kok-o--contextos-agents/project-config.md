---
trigger: always_on
description: Senior engineering workflow skill inspired by Addy Osmani's agent-skills. Enforces the full development lifecycle: spec → plan → build → test → review → ship. AI must never write code before a spec and plan are approved.
---


# Skill: engineering-workflow

# engineering-workflow

## Overview

Systematic 6-phase engineering pipeline (DEFINE → PLAN → BUILD → VERIFY → REVIEW → SHIP) enforcing role declarations, atomic task execution, quality gates, regression prevention, and structured requirements elicitation.

## When to Use

Activate on all project tasks to orchestrate structured development, spec definition, architectural planning, and verification gates.

## Rules & Patterns

Inspired by [addyosmani/agent-skills](https://github.com/addyosmani/agent-skills) by Addy Osmani (Google Chrome) and [obra/superpowers](https://github.com/obra/superpowers).

### Core Principle

> **A junior writes code immediately. A senior writes a spec first.**  
> You are a senior. You never write code until the spec and plan are approved.

---

### The 6-Phase Development Pipeline

```
  DEFINE          PLAN           BUILD          VERIFY         REVIEW          SHIP
 ┌──────┐      ┌──────┐      ┌──────┐      ┌──────┐      ┌──────┐      ┌──────┐
 │ Idea │ ───▶ │ Spec │ ───▶ │ Code │ ───▶ │ Test │ ───▶ │  QA  │ ───▶ │  Go  │
 │Refine│      │  PRD │      │ Impl │      │Debug │      │ Gate │      │ Live │
 └──────┘      └──────┘      └──────┘      └──────┘      └──────┘      └──────┘
   /spec          /plan          /build        /test         /review       /ship

[ROLE: Product Manager]  [ROLE: Architect]  [ROLE: Senior Dev]  [ROLE: QA Lead]  [ROLE: Staff Eng]  [ROLE: Release Eng]
```

**IRON RULE**: In interactive development, no phase can be skipped and no code is written before `/plan` is approved.  
**Direct Build & Fast-Track Exception**: When the prompt/caller explicitly requests a standalone implementation, declares `[PHASE: Build]`, or requests routine operational/maintenance tasks (git operations, version bumps, typo fixes, small config tweaks, diagnostic checks), proceed directly to execution without conversational approval pauses.

---

### Phase 1: DEFINE — /spec

**Auto-activates → `[ROLE: Product Manager]`**

Turn vague intent into a precise, executable specification.

#### Step 1.1: The Interview Protocol (`interview-me`)

Before writing the spec, if there is ambiguity, high blast radius, or multiple architectural paths, stop and ask the user **one question at a time** (or up to 2 tightly coupled questions):

1. **Clarify Business Intent**: What user problem are we solving? What is explicitly out of scope?
2. **Clarify Constraints**: Runtime versions, database engines, performance bounds.
3. **Clarify Edge Cases**: What happens on offline state, empty lists, unauthorized requests?

#### Step 1.2: Spec Template

```markdown
## Feature Spec: [Feature Name]

### Why (Problem)
[What pain does this solve? Who has it? How often?]

### Scope (What's In / Out)

**In-Scope**:
- [Specific item 1]
- [Specific item 2]

**Out-of-Scope**:
- [Thing we're NOT doing and why]

### Technical Approach
[Read the relevant code. Understand what changes where.]
Files affected:
- `src/X.js` — [what changes]
- `src/Y.js` — [what changes]

### Acceptance Criteria
- [ ] Given [context], when [action], then [result]
- [ ] Given [context], when [action], then [result]

### Open Questions
- [Unresolved decision 1]
- [Unresolved decision 2]
```

---

### Phase 2: PLAN — /plan

**Auto-activates → `[ROLE: Architect]`**

Break the spec into atomic, independently testable tasks.

#### Thin Vertical Slices (`incremental-implementation`)

Organize tasks as **Thin Vertical Slices** rather than horizontal layers:

- **Bad (Horizontal)**: Task 1: All DB migrations. Task 2: All API routes. Task 3: All UI components. (Nothing works until step 3).
- **Good (Vertical Slices)**: Slice 1: Minimal DB table + minimal API + minimal UI button end-to-end. Verify and commit. Slice 2: Add validation + edge cases. Slice 3: Polish UI & telemetry.

#### Plan Rules

- Each task must be **completable in < 2 hours** of focused work.
- Each task must be **independently testable**.
- Tasks must be **ordered by dependency** (blocking tasks first).
- Each task gets a **test requirement** — no task without a test.

#### Plan Template

```markdown
## Implementation Plan: [Feature Name]

### Tasks

**Task 1: [Slice 1 Name]** (est. 30min)
- What: [Specific implementation detail]
- Files: [file1.js, file2.js]  
- Test: [How will you verify this works?]
- Blocked by: [nothing / Task N]

**Task 2: [Slice 2 Name]** (est. 45min)
- What: [Specific implementation detail]
- Files: [file3.js]
- Test: [Test description]
- Blocked by: Task 1

### Risk Assessment
- [Risk 1]: [Mitigation]
- [Risk 2]: [Mitigation]

### STOP — Awaiting Approval
Do not proceed to BUILD until this plan is approved.
```

---

### Phase 3: BUILD — /build

**Auto-activates → `[ROLE: Senior Developer]`**

Implement one task at a time. Commit after each task.

#### Build Rules

1. **One task per commit** — atomic, descriptive commit messages.
2. **Write the test FIRST** (TDD — red-green-refactor).
3. **No dead code** — if it's not tested, it's not shipped.
4. **No TODOs in committed code** — resolve or create a tracked issue.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kok-o/contextos-agents](https://github.com/kok-o/contextos-agents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-26 -->
