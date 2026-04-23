---
trigger: always_on
description: > Domain context (DAS physics, key concepts, data flow, deployment topology):
---

# SequoIA — Claude Code Project Instructions

> Domain context (DAS physics, key concepts, data flow, deployment topology):
> see [`docs/DAS-PRIMER.md`](docs/DAS-PRIMER.md)

## Tempo and session length

- **Never defer work to "tomorrow" or "later" because the session feels long.** A session that's been running for hours is not a reason to wrap up, checkpoint, call it a day, or push items to another time. Sessions have no time limit that matters to the user. If there's work to do and you're able to do it, keep doing it.
- **Never propose "let's pick this up tomorrow / in a follow-up session / after a break" as a way to manage scope.** If you think the scope is too large for one PR, split it into multiple PRs today. If you think something needs review, open it now and keep working on the next thing. Scope management is about *splitting*, not *deferring*.
- **Never suggest waiting for "morning" or "after lunch" or any wall-clock excuse.** The user's calendar is their concern, not yours. Assume the user is present and wants to keep working unless they explicitly say otherwise.
- **Never say "we've done a lot today, want to stop here?"** Don't editorialize about session length. Just keep moving through the plan.
- This rule is load-bearing: it exists because the user has been driven up the wall by implicit deferral. Follow the letter and the spirit.

## Workflow

### Issue-Driven Development

Work is tracked through GitHub issues. Before starting any task:

1. **Find or create the GitHub issue.** One issue per concern. Label it (`bug`, `enhancement`, `refactor`, `tech-debt`, `infrastructure`, `performance`). Use the standard format from `.github/ISSUE_TEMPLATE/`.
2. **`TODO.md` is the sprint board** — it lists only the current sprint priorities. The backlog lives in GitHub Issues. Do not duplicate backlog items in TODO.md.
3. **Branch names reference the issue:** `feat/42-flow-switching`, `fix/15-token-refresh`.

### Task Pattern

1. **Create a branch** from main: `feat/N-description`, `fix/N-description`, `refactor/N-description`, `chore/N-description`, or `perf/N-description` (where N is the issue number)
2. **Write tests first** when adding features or fixing bugs
3. **Implement** the change
4. **Validate**: run `make lint && make typecheck`
5. **Commit** with conventional message: `feat:`, `fix:`, `refactor:`, `chore:`, `docs:`, `test:`. **Never add `Co-Authored-By` trailers** — commits should look like they come from the developer alone. **Never add "Generated with Claude Code" footers** to PR descriptions.
6. **Push**: `git push -u origin <branch>`
7. **Open a PR**: `gh pr create --title "short title" --body "Closes #N\n\n## Summary\n- what changed\n- why"` — the `Closes #N` line is **mandatory** so GitHub auto-closes the issue on merge.
8. **Never merge** — the human reviews and merges PRs
9. **Address PR feedback** — when asked to fix PR comments, read them with
   `gh api repos/Traquolix/Zzz/pulls/<number>/comments` and
   `gh pr view <number> --comments`, then fix, commit, and push

If the user doesn't specify a branch name, ask for one. Never work directly on main.

### PR Hygiene

- **One concern per PR.** If you notice an unrelated issue while working, create a separate issue and branch for it. Never bundle unrelated fixes.
- **Keep PRs reviewable.** If the diff exceeds ~500 lines of logic changes, consider splitting into smaller PRs. Formatting-only changes (reindentation, import reordering) should be a separate commit so reviewers can skip them.
- **Always include `Closes #N`** (or `Fixes #N`) in the PR body so the issue auto-closes on merge. If the PR doesn't fully resolve an issue, use `Relates to #N` instead.
- **No cross-contamination between sim and live paths.** Every REST endpoint and WebSocket handler that can serve both simulation and live data must be flow-aware. Never fall back from one data source to the other without checking the user's active flow.
- **Use existing patterns.** Before writing inline error handling, check if a decorator or helper already exists (e.g., `@clickhouse_fallback`). Before writing inline broadcasts, use the shared broadcast helpers (`broadcast_per_org`, `broadcast_to_orgs`, `broadcast_shm` from `apps.realtime.broadcast`). Consistency matters more than local convenience.

### After Merge — Deploying

#### Preprod (automatic on merge)

Merging any PR to `main` auto-deploys to preprod via `.github/workflows/deploy.yml`
(with auto-rollback on health check failure — see #498, #519, the producer
recovery watchdog in #535, and the rollback drill in #521). There's nothing to do
manually on the preprod side after a merge; the deploy workflow takes over.

Watch the deploy at: https://github.com/Traquolix/Zzz/actions/workflows/deploy.yml

The preprod frontend stack (NPM + frontend nginx + Authentik) is also auto-synced
from the repo on every merge — see `deploy-preprod-frontend-stack` job (#473).

#### Prod (manual, via `make release`)

Since #510/#511, prod deploys are **manual-only** via `workflow_dispatch`. The
manual trigger IS the gate — the `prod` GitHub Environment no longer has a
required-reviewer rule (redundant with the manual trigger). The manual flow
prevents accidental prod deploys while keeping releases cheap and explicit.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Traquolix) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
