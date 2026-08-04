---
trigger: always_on
description: Instructions for Claude Code (and other AI coding agents) working on this repository.
---

# CLAUDE.md

Instructions for Claude Code (and other AI coding agents) working on this repository.

## What this is

`openronin` — a self-hosted AI developer agent. It watches repos, picks up issues, opens PRs, iterates on review feedback, and merges when ready. Architecturally it's a three-layer split:

1. **Director** *(opt-in, separate systemd unit)* — proactive PM layer. Reads a per-repo charter and decides what the project should work on next. Emits decisions (create issue, comment, approve merge) into a chat thread. **Never edits source files.** See `docs/DIRECTOR.md` and `src/director/`.
2. **Supervisor** (MIMO) — cheap classifier that routes lanes.
3. **Worker** (Claude Code) — the only engine allowed to mutate code.

The agent eats its own dog food — most lanes after the bootstrap landed via the bot's own pipeline. If you're reading this as Claude Code working on this repo, you're literally working on yourself.

## Lanes

| Lane | Trigger | Default engine | Output |
|---|---|---|---|
| **triage** | new issue/PR | `mimo` | classification + labels |
| **analyze** | issue with `openronin:do-it` (before patch) | `mimo` | `state: ready` w/ expanded reqs OR `needs_clarification` w/ questions |
| **patch** | analyze=ready | `claude_code` | new branch + commit + draft PR |
| **patch_multi** *(opt-in)* | same | coder + reviewer | two-pass with critique |
| **pr_dialog** | new feedback on owned PR | `claude_code` | threaded reply + iteration push |
| **conflict_resolve** *(opt-in)* | mergeable=false during auto-merge | `claude_code` | rebase + edit markers + force-push-with-lease |
| **auto_merge** *(opt-in)* | all replies addressed + CI green | — | merge + close |
| **deploy** *(opt-in)* | push to trigger_branch by bot | shell | run configured commands |

`pickLane` in `src/scheduler/worker.ts` is the routing source of truth. PRs go to `pr_dialog`, never `triage`. Terminal `pr_branches.status` values (`dirty`, `no_changes`, `guardrail_blocked`, `needs_human`, `error`) skip the router until manually unstuck.

## Critical rules

- **State outside the code tree.** SQLite, work-trees, reports, secrets, ssh keys live under `$OPENRONIN_DATA_DIR`. The repo itself is byte-for-byte clean of state.
- **Engine boundary.** Code mutations only via the Claude Code worker. MIMO never edits files. Don't blur this line.
- **No vendor lock-in.** All VCS via `VcsProvider`, all task sources via `TrackerProvider`. Don't import `@octokit/*` outside `src/providers/github*.ts`.
- **Per-repo workers.** `scheduler.tickDrain` fans out parallel drains, one per watched repo. A long task in repo A doesn't block repo B.
- **Graceful shutdown.** `scheduler.stop(timeoutMs)` waits for workers + tracked side-activities (deploys). The SIGTERM handler in `src/index.ts` awaits up to 90s.
- **Bot self-events.** Every bot post starts with `BOT_PREFIX` (env-overridable, default `🥷 openronin:`). Filter via `isBotMessage()` everywhere a comment list is processed.
- **Bot git identity** comes from `getBotIdentity()` / `setBotIdentity(workdir)` in `src/lib/git.ts` — env-overridable. Don't hardcode `openronin[bot]` in lanes; call the helper.
- **Idempotency.** Patch lane checks existing `pr_branches IN ('created','open')` BEFORE any acknowledgement comment.
- **Iteration counter** (pr_dialog) only bumps on **successful pushes**. Failed iterations don't burn the budget.
- **SQLite UTC parsing.** Always use `parseSqliteUtc()` from `src/lib/time.ts` when comparing DB cutoffs to API ISO timestamps. Bare `new Date(sqlite_text)` shifts by the server's local TZ.
- **Force-push.** Always pass an explicit `<branch>:<sha>` lease to `git push --force-with-lease`. Bare form fails with `(stale info)` after a single-branch clone.
- **Director boundary.** The Director (`src/director/`) is read-only on source code. It only writes to its own DB tables (`director_*`) and emits decisions that the existing lane infrastructure carries out. Don't have it call `runPatch` directly, don't have it touch worktrees. If you're tempted to skip the lane router from the Director, you're doing something wrong.
- **Charter-pin every Director decision.** Each row in `director_decisions` must reference the `charter_version` it was produced under. Never write a decision without one — the audit trail breaks.

## Project layout

```
src/
├── index.ts                CLI vs server vs director:run entry, graceful SIGTERM
├── director/               Director: types, charter, chat, decisions, budget, service entry (separate systemd unit)
├── server/                 Hono: admin (HTMX+Tailwind CDN), webhooks, healthz, /api, layout, admin-director
├── supervisor/             selectEngine + runJob (cost cap + RateLimited handling)
├── engines/                mimo (cost calc), claude-code (RateLimited detect), anthropic, multi-agent
├── providers/              vcs interface; github, gitlab; tracker interface; jira, todoist, telegram
├── lanes/                  review, analyze, patch, patch-multi, pr-dialog, conflict-resolve, deploy, messages
├── storage/                db (migrations), repos, tasks, runs, pr-branches, deploys, reports, admin-audit
├── scheduler/              cadence, queue (repoId filter), reconcile, worker (pickLane), index (per-repo workers)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [openronin/openronin](https://github.com/openronin/openronin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-30 -->
