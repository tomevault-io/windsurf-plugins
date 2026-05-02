---
trigger: always_on
description: When executing session plan waves via /go command — task sequencing, quality checks, progress tracking
---


# Wave Execution (Cursor Adaptation)

> **CRITICAL**: Cursor does NOT have an `Agent()` tool. All wave tasks execute **sequentially in the current Composer session**. You are both the coordinator AND the implementer. Agent dispatch (3-tier resolution: project > plugin > general-purpose) is NOT applicable — follow the role's agent-prompt patterns directly.

> **State directory**: All state files live in `.cursor/` (STATE.md, wave-scope.json).

## Pre-Execution

Before starting Wave 1:

1. `git status --short` — commit or stash any uncommitted changes
2. Capture baseline: `SESSION_START_REF=$(git rev-parse HEAD)` — keep for the whole session
3. Confirm the agreed plan is still valid
4. Read `persistence` from Session Config (default: `true`)

### Initialize STATE.md (if persistence enabled)

Create `.cursor/STATE.md` (`mkdir -p .cursor` if needed):

```yaml
---
schema-version: 1
session-type: feature|deep|housekeeping
branch: <current branch>
issues: [<issue numbers from plan>]
started_at: <ISO 8601 timestamp>
status: active
current-wave: 0
total-waves: <from session plan>
---
```

```markdown
## Current Wave

Wave 0 — Initializing

## Wave History

(none yet)

## Deviations

(none yet)
```

## Wave Loop

For each wave in the session plan:

### Step 1: Write Scope Manifest

Write `.cursor/wave-scope.json` before each wave. Set `enforcement: warn` — Cursor's afterFileEdit hook is post-hoc (warns only, cannot block):

```json
{
  "wave": 1,
  "role": "<role>",
  "enforcement": "warn",
  "allowedPaths": ["<union of all task file scopes for this wave>"],
  "blockedCommands": ["rm -rf", "git push --force", "DROP TABLE", "git reset --hard", "git checkout -- ."]
}
```

Role-specific scope rules:
- **Discovery**: `allowedPaths: []` — READ-ONLY, do not edit files
- **Quality Phase 1 (Simplification)**: production files changed this session only (no test files)
- **Quality Phase 2 (Tests)**: test file patterns only (`**/*.test.*`, `**/*.spec.*`, `**/__tests__/**`)

Validate with: `bash "$CURSOR_RULES_DIR/../scripts/validate-wave-scope.sh"` — fix JSON if it exits 1.

### Step 2: Execute Tasks Sequentially

For each task in the wave:
1. Read the task description and acceptance criteria
2. Implement the task fully (you are the implementer)
3. Report status after each task:
   ```
   Task "[description]": done|partial|failed — [1-line summary]
   ```
4. If stuck, note the blocker and move to the next task

**DO NOT commit during wave execution** — `/close` handles commits.

### Step 3: Run Incremental Quality Checks

After ALL tasks in the wave complete:

| Role | Quality check |
|------|--------------|
| Discovery | None (read-only) |
| Impl-Core | typecheck + test changed files |
| Impl-Polish | typecheck + test changed files + integration verification |
| Quality | Full Gate: typecheck + test + lint — all must pass |
| Finalization | `git status` — verify clean state |

### Step 4: Quality Wave — Simplification Pass

At the START of the Quality wave, before running tests:

1. `git diff --name-only $SESSION_START_REF..HEAD` — list files changed this session
2. Filter to production files (exclude `*.test.*`, `*.spec.*`, `__tests__/`). If none, skip.
3. Review each production file for AI-generated patterns and apply targeted cleanup:
   - Remove unnecessary try-catch around non-throwing operations
   - Delete over-documentation (params that restate the name, returns that say "the result")
   - Replace re-implemented stdlib functions with standard alternatives
   - Simplify redundant boolean logic (if/else returning true/false, double negation)
   - **Do NOT change functionality**
4. After simplification, run Full Gate quality checks (typecheck + test + lint)

**Note:** Session-reviewer cannot be dispatched as a subagent on Cursor. Quality review is deferred to session-end Phase 1.8. In-wave quality relies on incremental checks + this simplification pass.

### Step 5: Update STATE.md (if persistence enabled)

After each wave:

1. **Frontmatter**: update `current-wave` to the completed wave number
2. **`## Current Wave`**: replace with next wave info
3. **`## Wave History`**: append:
   ```
   ### Wave N — <Role>
   - Task "[description]": done|partial|failed — [files changed] — [1-line note]
   ```
4. **`## Deviations`**: if plan was adapted, append:
   ```
   - [<ISO timestamp>] Wave N: <what changed and why>
   ```

### Step 6: Capture Wave Metrics (if persistence enabled)

Record after each wave:
- `wave_number`, `role`, `agent_count` (always 1 on Cursor), `files_changed`
- Per-task results: `{description, status: done|partial|failed}`
- `quality_check`: pass/fail/skipped

This data is written to `.orchestrator/metrics/sessions.jsonl` by session-end.

### Step 7: Report Wave Status

```
## Wave [N] ([Role]) Complete

- Task 1: [done/partial/failed] — [1-line summary]
- Task 2: [done/partial/failed] — [1-line summary]
- Tests: [passing/failing] | TypeScript: [0 errors / N errors]
- Adaptations for Wave [N+1] ([NextRole]): [none / list changes]
```

### Step 8: Adapt Plan (if needed)

- **On track**: proceed to next wave as planned
- **Minor issues**: add fix tasks to the next wave
- **Major blocker**: STOP — inform the user, propose revised plan

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Kanevry/session-orchestrator](https://github.com/Kanevry/session-orchestrator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
