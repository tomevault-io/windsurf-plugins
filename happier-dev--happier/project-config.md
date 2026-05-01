---
trigger: always_on
description: When Context7 is required for a round:
---


# Context7 evidence markers required when post-training packages are used

## Body
When Context7 is required for a round:
- Create a `context7-<package>.txt` marker per detected package in the round evidence directory.
- Notes in task/QA text are not accepted as evidence; use marker files only.

Reference: `guidelines/shared/CONTEXT7.md`

---
id: RULE.EVIDENCE.COMMANDS_VIA_TASKS_READY
title: Automation evidence must be captured via tasks/ready
category: validation
blocking: false
contexts: []
origins:
- core
source:
  file: guidelines/shared/VALIDATION.md
---

# Automation evidence must be captured via tasks/ready

## Body
Capture automation evidence through guarded tooling:
- Use the Edison guard/runner to generate command outputs and evidence markers.
- Do not paste manual command output into docs as a substitute for evidence files.

Reference: `guidelines/shared/VALIDATION.md`

---
id: RULE.EVIDENCE.ROUND_COMMANDS_REQUIRED
title: Preset-required command evidence must be captured
category: validation
blocking: false
contexts: []
origins:
- core
source:
  file: guidelines/shared/VALIDATION.md
---

# Preset-required command evidence must be captured

## Body
Before executing validators, ensure preset-required command evidence exists and passes for the current repo fingerprint:
- Inspect: `edison evidence status <task-id> [--preset <preset>]`
- Capture: `edison evidence capture <task-id> [--preset <preset>]`
- Evidence is snapshot-based (shared across rounds/tasks when the repo fingerprint is unchanged). Do not redirect output into files manually.

Reference: `guidelines/shared/VALIDATION.md`

---
id: RULE.EVIDENCE.ROUND_IMMUTABLE_APPEND
title: Evidence rounds are append-only
category: validation
blocking: false
contexts: []
origins:
- core
source:
  file: guidelines/shared/VALIDATION.md
---

# Evidence rounds are append-only

## Body
Evidence is append-only:
- Each re-run creates a new round directory; do not overwrite previous evidence.
- Never edit old evidence outputs to “make it look green”; rerun commands.

Reference: `guidelines/shared/VALIDATION.md`

---
id: RULE.QA.NO_DUPLICATE
title: No Duplicate QA Briefs
category: validation
blocking: false
contexts: []
origins:
- core
source:
  file: guidelines/shared/VALIDATION.md
---

# No Duplicate QA Briefs

## Body
Do not create duplicate QA briefs:
- Before creating a QA, search for an existing QA for the same task.
- If one exists, reuse it (move it through states), don’t create a new file.

Reference: `guidelines/shared/VALIDATION.md`

---
id: RULE.QA.PAIR_ON_WIP
title: Create QA brief when task enters wip
category: validation
blocking: false
contexts: []
origins:
- core
source:
  file: guidelines/orchestrators/SESSION_WORKFLOW.md
---

# Create QA brief when task enters wip

## Body
Pair QA early:
- As soon as a task enters wip, create (or move) its QA brief into waiting and add both to session scope.
- Do not defer QA creation until validation time.

Reference: `guidelines/orchestrators/SESSION_WORKFLOW.md`

---
id: RULE.QA.ROUND_HISTORY
title: QA Round History On Rejection
category: validation
blocking: false
contexts: []
origins:
- core
source:
  file: guidelines/shared/VALIDATION.md
---

# QA Round History On Rejection

## Body
On any rejection:
- Append a new “Round N” section to the QA brief summarizing findings and linking evidence.
- Create a new round evidence directory; never overwrite prior rounds.

Reference: `guidelines/shared/VALIDATION.md`

---
id: RULE.QA.WAITING_TO_TODO_TASK_DONE
title: QA waiting→todo allowed only when task is done
category: validation
blocking: false
contexts: []
origins:
- core
source:
  file: guidelines/shared/VALIDATION.md
---

# QA waiting→todo allowed only when task is done

## Body
Promotion gating:
- QA may move from waiting → todo only when the task is in done.
- If QA is “todo” while task is not done, treat it as a state mismatch and correct it.

Reference: `guidelines/shared/VALIDATION.md`

---
id: RULE.VALIDATION.BUNDLE_APPROVED_MARKER
title: Bundle Approved Marker Required
category: validation
blocking: true
contexts:
- validation
- transition
origins:
- core
source:
  file: guidelines/shared/VALIDATION.md
---

# Bundle Approved Marker Required

## Body
Bundle approval must be explicitly recorded:
- After ALL blocking validators approve, ensure the configured “bundle approved” marker exists in the round evidence directory.
- Promotions that require approval must fail-closed if the marker is missing or indicates not approved.

Reference: `guidelines/shared/VALIDATION.md`

---
id: RULE.VALIDATION.BUNDLE_FIRST
title: Bundle-First Validation Policy
category: validation
blocking: false
contexts:
- validation
origins:
- core
source:
  file: guidelines/shared/VALIDATION.md
---

# Bundle-First Validation Policy

## Body
Always validate from a bundle/manifest first:
- Generate the bundle/manifest before launching any validator.
- Validators must review only what the bundle lists (task(s), QA, evidence paths, diffs).

Reference: `guidelines/shared/VALIDATION.md`

---
id: RULE.VALIDATION.CONCURRENCY_CAP
title: Respect validator concurrency cap and batch overflow
category: validation
blocking: false
contexts: []
origins:
- core
source:
  file: guidelines/shared/VALIDATION.md
---


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [happier-dev/happier](https://github.com/happier-dev/happier) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
