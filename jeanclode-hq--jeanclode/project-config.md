---
trigger: always_on
description: Handles reach the container as `JEANCLODE_NOTIFY_USERS` (JSON array), resolved
---

# Jeanclode

Self-hosted autonomous coding agent. Listens to Sentry errors, GitHub, and GitLab events, then runs purpose-built multi-agent workflows on the Claude Agent SDK to triage and fix issues — opening PRs automatically without human intervention.

## Project Structure

- `backend/` - FastAPI backend — webhook ingestion, queue dispatch, tenant management, real-time SSE
- `cli/` - Python CLI (`jeanclode`) — thin runner that drives the multi-agent pipeline via Claude Agent SDK
- `security-proxy/` - Sidecar that injects credentials into agent HTTP traffic without exposing them to the subprocess
- `frontend/` - Nuxt 4 dashboard — workspace, integrations, live execution feed, leaderboards
- `packages/api-types/` - Shared TypeScript SDK generated from the backend OpenAPI schema
- `website/` - Nuxt marketing site + docs and blog (`website/content/`)

Workflows live under `cli/src/workflows/`:
- `sentry_fix/` — multi-phase pipeline: fetch → parallel triage (triage *is* the planner: it picks the target repo(s) and writes the fix plan) → synthesis (grouping only, so one root cause is one MR) → fix → CI-gate and label; a group's fix can span several repos, each with its own worktree and PR (opened ready, never draft) before the fixer runs, on one shared branch. Dispatch (ADR-006) partitions by `(sentry_org, git_org)` — each pair batches on the Sentry org's `batch_window`/`batch_size`, the **git org is the concurrency perimeter** (one fix batch per git org at a time; other git orgs run in parallel), and an opt-in per-org merge gate holds the next batch until the previous one's PRs land. The backend persists each PR the run reports and links it to the execution (`execution_pull_requests`), which drives the issue's dashboard status and the merge gate.
- `code_review/` — 7-agent pipeline: IssueExplorer → 2× Analyzer (parallel) → Synthesizer → Deduplicator → FactChecker → Guardrail → Styler; posts inline comments on GitHub PRs / GitLab MRs. On a bot-opened PR it either posts the `@jeanclode-bot` follow-up sweep (findings remain → another loop round) or, on LGTM, the ready notice that @-mentions the org's notify list — see "Ready notice" below
- `pr_summary/` — Summarizer → Parser, with a File Summarizer started 3s after the Summarizer (it shares the Summarizer's system prompt and output schema so it reads its prompt cache) → writes the PR/MR description plus a collapsed "Changes per file" File | Content table (one line per file; the file list comes from the diff, not the model); the summarizer carries over any link the old description had (Sentry issue, related MR, ticket) since it replaces that description wholesale
- `issue_resolve/` — triage → fix → open PR(s); triage explores the codebase itself and hands findings straight to the fixer (no separate plan stage); supports GitHub issues and GitLab issues/work_items; when a fix needs one or more linked repos beyond the issue's own, each gets a worktree and — once the fixer actually pushes to it — its own PR, opened lazily rather than speculatively. Which repos are available to it is resolved by `related_repos` (see backend/CLAUDE.md): repo groups, the org's always-include list, and the subgroup pack
- `jeanclode_respond/` — handles `@jeanclode-bot` mentions; planner either `route`s (defers to an existing pipeline — review/summary/resolve) or `handle`s the ask directly via Bash/git/`gh`/`glab`, judgment-governed rather than an enumerated whitelist. Two deterministic post-turn checks against provider state (never the planner's own account): the branch tip moved → re-attach `jeanclode:review`; nothing pushed but the last open thread closed → the ready notice, since that round of the loop converged with no re-review to come
- `_smoke/` — the `echo` workflow, an internal smoke test (no external API calls)

Every agent that reads `.context/diff` sees noise files (lockfiles, minified bundles, assets, snapshots, generated code) as a header plus a `+/-` count stub, never their content — `prepare_diff` in `cli/src/adaptors/diffn.py` owns that list.

## Ready notice

A bot-opened PR/MR runs a loop: review posts findings → the `@jeanclode-bot`
follow-up sweep dispatches respond → respond fixes and pushes (which
re-attaches `jeanclode:review`) → review runs again. It exits when review
finds nothing and posts LGTM.

Once that loop converges, the people a tenant picked in the git integration
page (org settings → `notify.on_ready`, stored as provider identity ids) get
@-mentioned in a comment. Two places emit it, because the loop has two exits:

* `code_review` on LGTM — the review came back clean.
* `jeanclode_respond` when a turn pushed nothing but resolved the last open
  thread — every finding was a false positive, so no push, no relabel, no
  re-review, and LGTM would never arrive.

Both go through `post_ready_notice`, which is post-once per PR/MR via an
HTML-comment marker (`src/runtime/notify.py`) so whichever exit fires second
stands down. It is a comment and not the description on purpose: `pr_summary`
rewrites descriptions wholesale, so mentions placed there don't survive.

Handles reach the container as `JEANCLODE_NOTIFY_USERS` (JSON array), resolved
backend-side by `add_notify_to_inputs` from identity ids against the org

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jeanclode-hq/jeanclode](https://github.com/jeanclode-hq/jeanclode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-16 -->
