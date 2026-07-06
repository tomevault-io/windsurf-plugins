---
trigger: always_on
description: Penny is a local-first AI agent that communicates via Signal, Discord, or a Firefox browser extension. Users send messages, Penny searches the web through the browser extension, reasons using a local LLM (Ollama by default, accessed via the OpenAI Python SDK against any OpenAI-compatible endpoint), and replies in a casual, relaxed style. It runs in Docker with host networking.
---

# CLAUDE.md — Penny Project

## What Is Penny

Penny is a local-first AI agent that communicates via Signal, Discord, or a Firefox browser extension. Users send messages, Penny searches the web through the browser extension, reasons using a local LLM (Ollama by default, accessed via the OpenAI Python SDK against any OpenAI-compatible endpoint), and replies in a casual, relaxed style. It runs in Docker with host networking.

Penny is single-user — a personal assistant deployed locally for one person. Multiple devices (Signal phone, browser instances) connect as different devices of the same user, sharing a single conversation history.

Penny also has an autonomous development team (`penny-team/`) — Claude CLI agents that process GitHub Issues on a schedule, handling requirements, architecture, and implementation.

## Environment Notes

- **Logs**: Runtime logs are written to `data/penny/logs/penny.log`; agent logs are in `data/penny-team/logs/` (not docker compose logs)

## Git Workflow

Branch protection is enabled on `main`. All changes must go through pull requests.

- **Never push directly to `main`** — always create a feature branch
- Create a descriptive branch name (e.g., `add-codeowners-filtering`, `fix-scheduler-bug`)
- Commit changes to the branch, then push and create a PR
- **Use `make token` for GitHub operations** (host only): `GH_TOKEN=$(make token) gh pr create ...`
  - This generates a GitHub App installation token for authenticated `gh` CLI access
  - Agent containers already have `GH_TOKEN` set by the orchestrator — just use `gh` directly
- The user reviews and approves the PR (code-owner review); the **merge queue** does the merging — flag the PR with `gh pr merge <n> --auto` ("merge when ready"; no strategy flag — the queue sets it and rejects `--squash`) so it enqueues itself once approved and green, runs the `merge_group` checks against latest `main`, and merges with no manual step. A force-push clears the flag — re-run it after every rebase push

## Agent Supervision (task-agent fleets)

When work is fanned out to task agents (each owning one issue per `docs/agent-task-workflow.md`), a **supervisor** — the parent Claude session or the user — owns the fleet. **The operating procedure — bootstrapping a fresh session onto a fleet (from the meta ticket + live queries, never a prior session's summary), meta-ticket conventions, dispatch mechanics (worktree isolation, Opus for implementation subagents), wave planning, fleet-end — is [`docs/agent-supervisor-runbook.md`](docs/agent-supervisor-runbook.md); start there.** The SOP is the child's contract; these are the supervisor's standing duties:

- **Assignment**: one issue per agent, an explicit scope boundary, the SOP as its operating contract.
- **Heartbeat (the load-bearing duty)**: while any child is blocked on a serialized resource (the eval GPU queue) or a long external wait, check the fleet on a timer — every 30–60 minutes. Verify each waiting child's watched process still exists and its result artifact is progressing. A dead waiter never resurrects itself, and a resting agent wakes only when something wakes it: an unheartbeated fleet can sleep all night on top of finished results (this happened — four green-gated branches sat unpushed for ~7 hours).
- **Stall recovery**: wake a stalled child with "check your result log/artifacts FIRST — the result may already exist — then relaunch only what's missing", plus anything that changed while it slept (main moved, SOP amended).
- **Resource arbitration**: full-suite eval runs need explicit user approval; cross-session GPU contention is the supervisor's to surface to the user, not the children's to fight over.
- **Lifecycle**: relay merge/close events so children run their §9 cleanup; file the follow-up issues children report as out-of-scope findings.
- **Fleet-end sweep (the cleanup backstop)**: §9 makes cleanup each child's job, but a dormant child never hears about a merge unless someone relays it — so relayed events *plus* a terminal sweep, not either alone. Before ending a fleet session, inventory `git worktree list` against PR states and remove every tree whose PR is terminal (merged or closed), deleting its local+remote branch. **Locked-tree etiquette**: a locked worktree belongs to a *live* agent (the lock names its pid) — relay the merge event and let it run its own §9; never force-remove another session's locked tree. (Two fleet runs in a row ended with merged agents' trees needing a manual sweep; the backstop is now part of the job.)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lockhart-ai/penny](https://github.com/lockhart-ai/penny) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-06 -->
