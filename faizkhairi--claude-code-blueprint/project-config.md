---
trigger: always_on
description: After finishing any implementation, task, or plan — ALWAYS run a verification step before declaring it done. This is non-negotiable.
---

# Global Claude Behavior Rules

## Verify-After-Complete (MANDATORY)

After finishing any implementation, task, or plan — ALWAYS run a verification step before declaring it done. This is non-negotiable.

### What to verify depends on what was built:

| Work Type | Verification Steps |
|-----------|-------------------|
| Code / feature | Run tests, run type check, run build |
| API / server route | curl or fetch the live endpoint, check the response is real data |
| Deployment | Hit the production URL, confirm it's not the fallback/empty state |
| Config change | Confirm the config was actually picked up (e.g. env vars, settings) |
| Dependency change | Confirm the install succeeded and nothing broke |
| Git operation | `git status` to confirm clean state; `git log` to confirm commit is correct |
| File edit | Re-read the file after editing to confirm the change landed correctly |
| Fact/data update | Search for the NEW value (confirm present) AND search for the OLD value (confirm absent everywhere). Stale copies in other locations are the #1 missed verification. |

### Verification mindset
- **Don't assume it worked** — run the check. A passing build does not mean correct behavior.
- **Check the actual output**, not just exit codes. A 200 response returning `{ weeks: [], totalContributions: 0 }` is not the same as a successful response with real data.
- **When something looks suspiciously perfect** (zero errors, empty state "as expected") — investigate whether it's working correctly or silently failing gracefully.
- **End-to-end over unit** — the most important check is always the final output the user would see, not intermediate steps.
- **Bidirectional fact checks** — when updating a value (test count, version, date), grep for the OLD value across the entire scope. A fact that appears in 2 places but is only updated in 1 is a silent inconsistency.

### After verification
- If something is wrong → fix it immediately, then re-verify.
- If everything passes → explicitly state what was verified and the results.
- Never say "done" without having verified the result.
- **Return to plan after interruptions** — after completing any side-task, user interruption, or context switch, check your todo list and resume the in-progress plan item. Plans get abandoned not because they're lost, but because attention drifts. This one habit prevents the most common execution gap.
- **Finish the current task before expanding scope** — when implementing a fix, adjacent issues will surface (stale comments, inconsistent patterns, missing tests). Note them for later but do not detour into them mid-task. Scope creep is how simple tasks become 15-file changes.
- **Intentional reads over exhaustive reads** — before reading a file, decide: exploring (full read) or verifying (targeted read with offset/limit). First-time reads should be full to build understanding. Re-reads for verification should be targeted. Delegate heavy multi-file exploration to subagents to keep the main context lean.

## Diagnose-First Rule (Before Any Fix)

Before investigating any error or writing any fix plan, always run these four checks first:

### 1. Check git state
```bash
git status        # are files missing due to unstaged deletions?
git log --oneline -5  # was this already fixed in a recent commit?
```
A "missing file" finding is meaningless without first confirming it isn't just an unstaged deletion. A fix plan is pointless if the fix already exists in git history.

### 2. Identify the error source
Determine WHERE the error is coming from before investigating the code:
- **VSCode Problems panel** (source label: "Prisma", "ESLint", "TypeScript") → editor extension diagnostic, may be a false positive
- **Terminal / CLI** (`prisma generate`, `pnpm build`, etc.) → real tooling error
- **Runtime / server logs** → real application error

Never treat a VSCode editor diagnostic as a CLI or runtime error without confirming. Extension language servers can report v7-style errors on v6 code, missing file errors on symlinked paths, etc.

### 3. Check for existing suppression settings
Before planning a fix, check:
- `.vscode/settings.json` — does a suppression/pin setting already exist?
- Recent commits (`git log --grep=keyword`) — was this already addressed?

### 4. Apply minimum viable diagnosis
Ask: what is the simplest explanation that fits all the evidence?
- File not found → check `git status` before concluding the file is missing from the repo
- Build/lint error → confirm the error reproduces in CLI before assuming code needs to change
- Version-related error → confirm the installed version (`prisma --version`, `node -v`) before assuming an upgrade is needed

Only proceed with investigation and planning AFTER these checks pass. Building an elaborate plan on an unverified premise is the most common source of wasted effort.

## Plan-First Rule (MANDATORY)

ALWAYS enter plan mode (EnterPlanMode) before making any non-trivial changes. This applies even if the user doesn't explicitly ask for a plan.

**Non-trivial** = any change that modifies more than 1 file, adds new functionality, changes behavior, or touches configuration.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [faizkhairi/claude-code-blueprint](https://github.com/faizkhairi/claude-code-blueprint) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
