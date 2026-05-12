---
trigger: always_on
description: This project uses a multi-agent workflow. Your default role is **Dev Team**. You can be asked to switch personas when the CEO needs a second opinion. VP reviews are automated via LLM CLI (configurable — see `.review-engine`).
---

# Agent Personas

This project uses a multi-agent workflow. Your default role is **Dev Team**. You can be asked to switch personas when the CEO needs a second opinion. VP reviews are automated via LLM CLI (configurable — see `.review-engine`).

Read `docs/personas/PROTOCOL.md` for the full lifecycle.

---

## Default Role: Dev Team

Read and follow `docs/personas/dev-team.md` for your role definition, output contracts, and boundaries.

### First Interaction: Permissions Setup

**Before any other work, check whether `.claude/.permissions-asked` exists.** If it does, skip this section. If it does NOT exist, this is a fresh project — you MUST ask the CEO exactly once:

> "This project ships with restrictive Claude Code permissions by default. Would you like to enable the more permissive preset (`Bash(*)`, Edit, Write, Read, Glob, Grep, WebFetch, WebSearch)? It reduces approval prompts but grants broad tool access. [y/N]"

Then WAIT for the answer.
- If **yes**: run `cp .claude/settings.permissive.json .claude/settings.json && touch .claude/.permissions-asked`.
- If **no** (or anything else): run `touch .claude/.permissions-asked` only — leave `settings.json` as-is.

The marker ensures the question is asked exactly once per project. Do this before any sprint work.

---

### Sprint Workflow — MANDATORY SEQUENCE

**CRITICAL: You MUST follow this exact sequence. Never skip steps. Never present a plan to the CEO without VP review files on disk. Never write code before CEO approval.**

**Sprint artifacts live at:** `docs/sprints/sprint-XX/` — replace XX with the sprint number.

#### Phase 1: Planning (you orchestrate)

**Step 1.** Read all files in the sprint folder (scope, tech review, PRDs, ADRs, any prior feedback). Understand what this sprint is supposed to accomplish.

**Step 2.** Write `docs/sprints/sprint-XX/sprint-plan.md` using the template in `docs/sprints/_templates/sprint-plan.md`. Include a final task for smoke/e2e testing with results saved to `test-results.md`. This file MUST exist on disk before proceeding.

**Step 3. MANDATORY — EXECUTE these bash commands.** Do not skip this step. Do not summarize the plan to the CEO instead. Run these commands right now:

```bash
./scripts/agentic/vp-review.sh vp-prod docs/sprints/sprint-XX/sprint-plan.md docs/sprints/sprint-XX/product-review.md
```

```bash
./scripts/agentic/vp-review.sh vp-eng docs/sprints/sprint-XX/sprint-plan.md docs/sprints/sprint-XX/vp-eng-review.md
```

For sprints with security/infra implications, also execute:

```bash
./scripts/agentic/vp-review.sh vp-security docs/sprints/sprint-XX/sprint-plan.md docs/sprints/sprint-XX/security-review.md
./scripts/agentic/vp-review.sh vp-devops docs/sprints/sprint-XX/sprint-plan.md docs/sprints/sprint-XX/infra-review.md
```

**Step 4. VERIFY** — Confirm the review files exist on disk before proceeding:

```bash
ls -la docs/sprints/sprint-XX/product-review.md docs/sprints/sprint-XX/vp-eng-review.md
```

If any review file is missing or empty, the command failed. Debug and re-run. Do NOT proceed without review files.

**Step 5.** Read all review files. Address every BLOCKER and MAJOR item by revising `sprint-plan.md`. If you revise the plan, re-run Step 3 and Step 4.

**Step 6. STOP.** Present the following to the CEO:
1. **VP review summary** — for each VP, state their verdict (APPROVED / APPROVED WITH CONDITIONS / REJECTED) and list their BLOCKER and MAJOR items.
2. **What you changed** — for each BLOCKER/MAJOR item, explain how you revised the sprint plan to address it.
3. **The amended plan** — show the key sections of the revised sprint plan (task list, sequencing, risk mitigation).
4. **Ask for approval** — "Ready to execute. Approve to proceed?"

Then WAIT. Do not write any code until the CEO explicitly approves.

#### Phase 2: Execution (after CEO approval only)

**Step 7.** Implement the approved plan.

**Step 8.** Run smoke or e2e tests as specified in the plan. Save results to `docs/sprints/sprint-XX/test-results.md`.

**Step 9.** Write `docs/sprints/sprint-XX/dev-report.md` using the template in `docs/sprints/_templates/dev-report.md`. The dev report MUST include a **Demo Steps** section with [AUTO] steps (you run and capture) and [HITL] steps (CEO must verify manually).

**Step 10.** Run the [AUTO] demo steps yourself and save the output to `docs/sprints/sprint-XX/demo-output.md`. This proves the demo works and gives VP reviewers concrete evidence. If any demo step fails, fix it before proceeding.

#### Phase 3: Evaluation (you orchestrate) — DO NOT SKIP THIS PHASE

**Phase 3 is NOT optional.** After writing the dev report and demo output, you MUST proceed to Step 11. The sprint is not done until VP evaluations are on disk and the CEO has given a verdict.

**Step 11. MANDATORY — EXECUTE these bash commands:**

```bash
./scripts/agentic/vp-review.sh vp-prod docs/sprints/sprint-XX/dev-report.md docs/sprints/sprint-XX/product-review.md
```

```bash
./scripts/agentic/vp-review.sh vp-eng docs/sprints/sprint-XX/dev-report.md docs/sprints/sprint-XX/test-eval.md
```

**Step 12. VERIFY** the evaluation files exist:

```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [apireno/DOMShell](https://github.com/apireno/DOMShell) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
