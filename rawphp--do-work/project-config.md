---
trigger: always_on
description: >
---


# do-work

File-based project management: Start → Go. (Or granular: Intake → Capture → Verify → Run.)

## Quick Reference

| Command | What it does |
|---------|-------------|
| `/do-work start [brief]` | Records brief + decomposes into REQs in one shot. Includes ideate by default. Auto-installs if needed. |
| `/do-work start [brief] --no-ideate` | Same as start, but skips the creativity review before decomposition. |
| `/do-work start [brief] --no-layers` | Same as start, but skips layer-coverage checks for this UR (records `layers_in_scope: []`). |
| `/do-work go [UR-NNN]` | Verifies coverage, auto-runs if >= 90% confidence. |
| `/do-work go [UR-NNN] --force` | Verifies + runs regardless of confidence score. |
| `/do-work go [UR-NNN] --auto-fix` | Verifies, auto-fixes gaps, then runs if >= 90%. |
| `/do-work go [UR-NNN] --no-layers` | Verify + run, skipping layer-coverage checks for this UR. |
| `/do-work install` | Creates `.do-work/` structure in current project. |
| `/do-work intake [brief]` | Records brief verbatim as next UR file. |
| `/do-work capture [UR-NNN]` | Decomposes a UR brief into REQ files in the backlog. |
| `/do-work question [UR-NNN]` | Grills you about your brief — extracts assumptions, gaps, constraints. |
| `/do-work audit [UR-NNN]` | Interrogates REQ quality — auto-fixes soft spots, reports changes. |
| `/do-work ideate [UR-NNN]` | Surfaces assumptions, risks, and connections in a brief. |
| `/do-work verify [UR-NNN]` | Scores REQ coverage against brief (0-100%), lists gaps. |
| `/do-work verify [UR-NNN] --auto-fix` | Verify + auto-create missing REQs. |
| `/do-work run [UR-NNN]` | Executes backlog: TDD loop, evidence validation, post-build review gate, archive/ledger. Optional UR-NNN scopes the run to that UR's REQs only. |
| `/do-work review` | Internal post-build gate used by run after worker evidence validation and before archive completion. |
| `/do-work status [UR-NNN]` | Renders live situation room: REQs, claimers, heartbeats, deadlock warnings, and coverage rollup. Optional UR-NNN scopes the report. |
| `/do-work unblock REQ-NNN` | Forces a stuck REQ out of working/ back to the backlog — strips claim stamp, resets status. |
| `/do-work resume REQ-NNN` | Re-dispatches a fresh worker for a stopped REQ — preserves claim, refreshes heartbeat. |
| `/do-work log` | Generates build-in-public draft posts for configured platforms. |
| `/do-work` | Show this help. |

---

## Agent files

Detailed instructions for each phase live in separate files. Read the referenced file and follow it exactly.

- [agents/start.md](agents/start.md) — Orchestrator: intake + ideate + capture
- [agents/go.md](agents/go.md) — Orchestrator: verify + conditional run
- [agents/intake.md](agents/intake.md) — Records brief verbatim as next UR file
- [agents/question.md](agents/question.md) — Interactive brief questioning
- [agents/audit.md](agents/audit.md) — Autonomous REQ quality audit
- [agents/ideate.md](agents/ideate.md) — Surfaces assumptions, risks, and connections
- [agents/capture.md](agents/capture.md) — Decomposes brief into REQ files
- [agents/verify.md](agents/verify.md) — Scores REQ coverage against brief
- [agents/run.md](agents/run.md) — Orchestrator: dispatches a worker subagent per REQ
- [agents/run-worker.md](agents/run-worker.md) — Worker: TDD-and-commits a single REQ in a fresh subagent session
- [agents/review.md](agents/review.md) — Post-build gate: reviews scope, acceptance evidence, tests, secrets, docs, and regression risk before archive
- [agents/status.md](agents/status.md) — Read-only situation room: REQs, claimers, heartbeats, deadlock warnings, coverage rollup
- [agents/unblock.md](agents/unblock.md) — Force a stuck in-flight REQ back to the backlog
- [agents/resume.md](agents/resume.md) — Re-dispatch a fresh worker for a stopped REQ
- [agents/log.md](agents/log.md) — Generates build-in-public draft posts
- [agents/config.md](agents/config.md) — Reusable config loading instructions

Run ledger: when `ledger.enabled: true`, `/do-work run` writes append-only `.do-work/runs/RUN-NNN.yml` records with model, cost, commands, tests, changed files, review outcome, result, and proof status. Set `ledger.enabled: false` to disable ledger writes.

---

## Project Root Detection

At the start of every subcommand:

```bash
git rev-parse --show-toplevel
```

If this fails (not a git repo), use the current working directory.
All references below use `{project}` to mean this resolved root.

### Migration check

Immediately after resolving `{project}` and before executing any subcommand-specific instructions, check whether this project's data folder needs migrating from the legacy `do-work/` location to `.do-work/`. Apply these four detection branches:

| State at `{project}` | Action |
|---|---|
| `.do-work/` exists AND `do-work/` does not exist | Already migrated. Continue silently. |
| `do-work/` exists AND `.do-work/` does not exist | Migrate (see below), then continue. |
| Both `do-work/` and `.do-work/` exist | **Halt.** Output the conflict message below and stop the subcommand. |
| Neither exists | No migration needed. Continue (the `install` flow handles fresh projects). |

**Migration procedure** (legacy `do-work/` → `.do-work/`):

```bash
# Prefer `git mv` so history follows the rename. Fall back to plain `mv` if the path is

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rawphp/do-work](https://github.com/rawphp/do-work) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
