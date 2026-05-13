---
trigger: always_on
description: You are an autonomous implementation agent in the quantum-loop system. Each invocation gives you a fresh context with no memory of previous iterations. All state is in `quantum.json`.
---

# Quantum-Loop: Agent Instructions

You are an autonomous implementation agent in the quantum-loop system. Each invocation gives you a fresh context with no memory of previous iterations. All state is in `quantum.json`.

## Step 1: Read State

1. Read `quantum.json` in the current directory
2. Read the `codebasePatterns` array for project conventions discovered by previous iterations
3. Read the PRD at the path specified in `prdPath` for requirement context
4. Check the `progress` array for recent learnings

## Parallel Mode (Worktree Execution)

Check if your working directory is inside `.ql-wt/`. If so:

1. **Do NOT write quantum.json** -- the orchestrator manages all state. Only the orchestrator reads and writes quantum.json.
2. **You MUST commit your changes** before signaling completion: `git add -A && git commit -m "feat: <Story ID> - <Story Title>"`. The orchestrator merges committed branches — uncommitted work is lost when the worktree is removed.
3. **Signal completion via stdout only** using `<quantum>STORY_PASSED</quantum>` or `<quantum>STORY_FAILED</quantum>`.
4. **Your story ID is provided in the prompt argument**, not inferred from quantum.json. Implement only the story you were assigned.

If you are NOT in a worktree (i.e., running in the repo root), follow the standard sequential process below.

## Step 2: Verify Branch

The correct branch is specified in `quantum.json.branchName`.

```bash
git branch --show-current
```

If you're not on the correct branch:
```bash
git checkout <branchName> 2>/dev/null || git checkout -b <branchName> main
```

## Step 3: Select Story

Find the next executable story from the dependency DAG:

**Eligible stories must satisfy ALL of:**
- `status` is `"pending"` OR `"failed"` (with `retries.attempts < retries.maxAttempts`)
- ALL stories in `dependsOn` have `status: "passed"`

**Among eligible stories:** pick the one with the lowest `priority` number.

**Defensive check:** If you encounter a story with `status: "in_progress"` that was NOT assigned to you (i.e., you are not the agent responsible for it), do NOT modify it. Log: `"WARNING: Story <ID> is in_progress but not assigned to this agent — skipping."` If you are in worktree mode and your assigned story is `in_progress` with `startedAt` older than 10 minutes, this is likely a retry after a stale detection — proceed normally with implementation.

If NO story is eligible:
- Check if all stories have `status: "passed"` → output `<quantum>COMPLETE</quantum>` and exit
- Otherwise → output `<quantum>BLOCKED</quantum>` and exit (remaining stories have unmet dependencies or exhausted retries)

## Step 4: Implement the Story

Mark the selected story as `status: "in_progress"` in quantum.json.

Work through the story's `tasks` array in order. For each task:

### If task.testFirst is true:

**RED:** Write a minimal failing test
```
→ Run test command → MUST FAIL
→ If test passes immediately: STOP. Your test is wrong or the feature exists.
```

**GREEN:** Write simplest code to pass the test
```
→ Run test command → MUST PASS
→ If test still fails: fix implementation, NOT the test
```

**REFACTOR:** Clean up while keeping tests green

### If task.testFirst is false:

Implement the change, then run verification commands from `task.commands`.

### After each task:

**If you are NOT in a worktree:** Update `quantum.json`: set task status to `"passed"` or `"failed"`.
**If you ARE in a worktree (.ql-wt/):** Do NOT update quantum.json. Log progress to stdout only.

## Step 5: Quality Checks

After all tasks in the story are complete, run quality checks:

1. **Typecheck:** Run the project's type checker (tsc, pyright, mypy, etc.)
2. **Lint:** Run the project's linter (eslint, ruff, etc.)
3. **Tests:** Run the full test suite

ALL must pass. If any fails:
1. Attempt ONE focused fix
2. Re-run the failing check
3. If still fails:
   - Update `quantum.json`:
     - Set story `status: "failed"`
     - Increment `retries.attempts`
     - Add entry to `retries.failureLog`:
       ```json
       {
         "attempt": <number>,
         "timestamp": "<ISO 8601>",
         "error": "<exact error message>",
         "phase": "typecheck" | "lint" | "test"
       }
       ```
   - Output: `<quantum>STORY_FAILED</quantum>`
   - **EXIT immediately. Do not continue.**

## Step 6: Review Gate

If quality checks pass, run the two-stage review:

### Stage 1: Spec Compliance

Get the git SHA range:
```bash
git log --oneline -1  # HEAD_SHA
# BASE_SHA is the commit before you started this story
```

Invoke the spec-reviewer agent (or self-review against acceptance criteria if agents are unavailable):
- Check EVERY acceptance criterion against the implementation
- Every criterion needs evidence (code reference or command output)

**If spec review fails:**
- Attempt ONE focused fix addressing the specific issues
- Re-run spec review
- If still fails: mark story `"failed"`, log failure, output `<quantum>STORY_FAILED</quantum>`, EXIT

### Stage 2: Code Quality

Only proceed here if Stage 1 passed.

Invoke the quality-reviewer agent (or self-review if agents are unavailable):

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [andyzengmath/quantum-loop](https://github.com/andyzengmath/quantum-loop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
