---
trigger: always_on
description: validates the LLM's structured result) qualifies `out["answer"]` using the
---

# Project agent memory

Wheelhouse - a portable, forkable IssueOps machine. Issues in this repo are a
human-in-the-loop decision queue for cross-repo OSS maintenance, driven entirely
by GitHub Actions. This file holds durable, project-intrinsic notes.

The name: a ship's wheelhouse is where the captain steers. This repo is where
you steer your open-source maintenance - what needs your hand surfaces as a card
and you make the call. (The product is "Wheelhouse"; the generic verb "triage"
still appears where it's plain English, e.g. "triage the queue".)

## Non-negotiable invariants

- **Portability / fork-and-own.** Never hardcode an owner or repo name in
  workflows or scripts. Owner is always `github.repository_owner` (env
  `GITHUB_REPOSITORY_OWNER`); the fleet + policy come from the single root file
  `wheelhouse.config.yml`. A fork on any account must work after editing only that
  file and adding the secrets.
- **Security.** Owner-gate every acting path (`sender == repository_owner`, plus
  optional `maintainer` override via `wheelhouse_core.py authorized`). Cross-repo
  actions use `FLEET_TOKEN`; everything that touches THIS repo's cards uses the
  default `GITHUB_TOKEN` (this is also what prevents the decision-handler from
  re-triggering itself - GitHub does not raise workflow events for
  GITHUB_TOKEN-authored activity). The fork-CI / pwn-request HOLD (exit 4 in
  `approve_ci`) must never be removed: approving fork CI that changes
  `.github/workflows`, `.github/actions`, or `action.yml(.yaml)` is held for
  manual review and fails closed. **Scan-time auto-approve is a STRICT SUBSET of
  the manual gate**: it shares the one `ci_safety` verdict and approves only what
  is provably safe (no risky files AND no `pull_request_target` posture, all reads
  fail closed), so it can never auto-clear anything the manual path would HOLD.

## Architecture

- **State lives in GitHub, not on disk.** Open issue = pending decision; closed =
  consumed. Labels are state (`needs-decision`, `pending-triage`, `processing`,
  `resolved`, `blocked`, `repo:*`, `kind:*`, `priority:*`). A hidden
  `<!-- wheelhouse-state: {...} -->` block in each card body carries
  `{repo, number, kind, head_sha, options}` plus the material fields
  `{comp, tests, priority}` (the latter three added so a refresh can cheaply and
  deterministically decide "did this target materially change?" - see "Card
  refresh" in Sharp edges). `options` is also material for refresh comparison,
  but is normalized as a sorted set so checkbox reordering alone does not
  refresh the card. The state block also carries `updated_at` unconditionally
  (populated for issue-triage items, empty for pr-review) - it is NON-material,
  existing purely as the issue-triage auto-triage cache key, mirroring how
  `head_sha` doubles as the pr-review cache key. Automatic triage (pr-review
  AND issue-triage) adds non-material cache fields such as
  `triaged_sha` and `triage_status`; those are deliberately outside
  `MATERIAL_FIELDS` so a triage result never changes classification or forces a
  card refresh. A held card also carries non-material `held: true` until its
  first auto-triage attempt publishes the normal decision controls. The state
  block also carries `render_version`, another
  non-material field alongside `triaged_sha`: it is a one-time re-render
  trigger stamped by `render()` (see "Card refresh" in Sharp edges) that exists
  purely so a display-only fix (e.g. the author `@mention` drop) propagates to
  already-open cards; it is never a `MATERIAL_FIELDS` member and never
  influences classification. `render_card.py` writes that marker, but
  `parse_state_block` also accepts the legacy `<!-- triage-state: ... -->`
  marker (cards rendered before the rename) - back-compat that must stay so a live
  queue keeps working. It also tolerates old `wheelhouse-state` cards that lack
  the material fields: a missing field reads as "unknown", so such a card is seen
  as changed exactly once and refreshes itself (backfilling the fields), then
  no-ops. The local lock/board/ledger from the original `triage.py`
  are intentionally dropped (replaced by Actions
  `concurrency` + issues/labels/comments).
- **Workflows:** `ingest` (dispatch/manual -> upsert a card), `decision-handler`
  (tick/slash/**plain-English** -> act on target -> consume card), `scan-backstop`
  (hourly scan -> reconcile: create/refresh/close - the primary keep-current path
  now that cards refresh on material change, render-version staleness, or a
  held-card publish trigger; safe to run hourly because reconcile is a full
  no-op when none of those triggers fires, and queues automatic PR or issue
  triage when the
  current revision (a PR's `head_sha`, or an issue's `updatedAt`) lacks a fresh
  `triaged_sha` cache; its "List open cards" step lists THIS repo's open cards via
  `gh api --paginate --slurp "repos/{owner}/{repo}/issues?..." | jq '...'` -
  `gh api --slurp` and `--jq` are mutually exclusive in the installed `gh` CLI, so
  the `--paginate --slurp` result (an array of per-page arrays) is piped into a
  standalone `jq` instead of passing `--jq` to `gh api` itself;
  `tests/test_workflow_lint.py` guards against this combination reappearing in

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kunchenguid/wheelhouse](https://github.com/kunchenguid/wheelhouse) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-05 -->
