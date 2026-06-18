---
trigger: always_on
description: Aragora is the control plane for multi-agent vetted decisionmaking across organizational knowledge and channels. It implements structured vetted decisionmaking through a society of heterogeneous AI agents. This document describes the agent system architecture.
---

# Aragora Agents

Aragora is the control plane for multi-agent vetted decisionmaking across organizational knowledge and channels. It implements structured vetted decisionmaking through a society of heterogeneous AI agents. This document describes the agent system architecture.

> **Operating Contract:** Autonomous CLI agents (Claude Code, Codex CLI, Factory Droid,
> Aider, the agent-bridge harnesses) working *in* this repository operate under
> [`docs/AGENT_OPERATING_CONTRACT.md`](docs/AGENT_OPERATING_CONTRACT.md) — the
> always-allowed / approval-required matrix, the "break unreleased branch behavior
> freely; never break main / public API / release flow / CI" rule, and main-red
> incident mode. That contract governs *how* agents execute against the repo. This
> document describes *what* agents are registered as runtime debate participants
> (the 43-agent registry).

## Worktree Autopilot (High-Churn Sessions)

When many agents are committing concurrently, use disposable worktrees with frequent reconciliation.

- Start sessions with `./scripts/codex_session.sh` (or `make codex-session`).
  This writes an active-session lock so background maintenance skips in-use worktrees.
- Do not delete side worktrees with raw `git worktree remove` plus `rm -rf`.
  Inspect/remove them with `python3 scripts/safe_worktree_cleanup.py` so active-session locks and open PR branches block accidental deletion.
- Auto-heal unexpected worktree/branch drift with:
  `python3 scripts/codex_worktree_autopilot.py ensure --agent codex --base main --reconcile --print-path`
- Prefer one-shot upkeep during rapid churn:
  `python3 scripts/codex_worktree_autopilot.py maintain --base main --strategy merge --ttl-hours 24`
- Reconcile managed sessions with:
  `python3 scripts/codex_worktree_autopilot.py reconcile --all --base main`
- Remove stale managed worktrees with:
  `python3 scripts/codex_worktree_autopilot.py cleanup --base main --ttl-hours 24`
- Optional macOS daemon: `make worktree-maintainer-install` for periodic background reconcile-only upkeep.

## Automation Operating Rules

For Codex-driven automations in this repo, default to maximum safe autonomy. Finish the bounded task when the next action is clear, and only stop when the remaining step is irreversible, human-gated, or materially unsafe.

- Use the shared automation merge contract in `docs/briefs/automation-merge-contract.md`.
  Before publishing local Codex app automation or Aragora boss-loop worker branches, run `bash scripts/automation_pr_preflight.sh origin/main HEAD` from the branch worktree, or run it against the explicit worker branch.
- Prefer execution over advice:
  verify the issue, make the smallest credible fix, validate it, commit it, push it, open the PR, and leave the inbox or memory handoff in the same run when the task is otherwise ready.
- Do not stop at the first blocked path:
  inspect `--help`, adapt to the actual helper interface, and try the next practical route before declaring a blocker.
- Use layered fallbacks:
  move between shell git/gh, MCP connectors, local repo inspection, and browser flows when one surface is degraded.
- Do not launch Factory/Droid in interactive Auto Off mode for normal Aragora work.
  Prompted Droid/Factory lanes should go through `scripts/tmux_session_launcher.sh`
  or `scripts/agent_bridge.py launch`, which route them through `droid exec --auto high`.
  Only use `ARAGORA_ALLOW_DROID_AUTO_OFF=1` for an explicit manual-debugging exception.
- Recover cleanly from partial failure:
  if publish or inbox delivery fails, still leave a clean committed branch or an exact handoff with the compare URL, blocker, and next action.
- Treat founder guidance as strong but verify it:
  if founder memory is fresh but imperfectly structured, verify the recommendation directly on `origin/main` instead of discarding it mechanically.
- Keep everything reversible:
  use disposable worktrees, branch-scoped commits, additive edits, and non-destructive cleanup. Never delete worktrees or branches with uncommitted changes, unique commits, or open PRs.
- Keep scope bounded:
  prefer measurable improvements on live paths over speculative breadth, and use Aragora itself when it improves decisions without dominating a small direct fix.
- Before lane work, check your operator-steering mailbox with
  `python3 scripts/read_operator_steering.py --lane-id <LANE_ID> --json` (or
  `--pr` / `--branch` when that is the only selector you know). If you read a
  message, write an outcome receipt with `--outcome obeyed|held|stale|superseded|blocked|completed`
  before mutating lane state. `_read_receipts/` is proof-of-read/outcome only;
  it is not an ack protocol, and top-level message files remain pending until a
  future explicit ack/move protocol exists.
- Active lanes must expose their next action and owner liveness. Claim/refresh
  lanes with `next_action`, `last_heartbeat_at`, and `last_steering_outcome`;
  long-running harnesses should also run `python3 scripts/agent_heartbeat.py`
  so other sessions can see owner_session, pid/thread hints, cwd, worktree,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [synaptent/aragora](https://github.com/synaptent/aragora) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
