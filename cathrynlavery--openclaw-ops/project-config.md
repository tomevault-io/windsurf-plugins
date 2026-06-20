---
trigger: always_on
description: Use when installing, configuring, troubleshooting, securing, or performing a health check on OpenClaw gateway setups — including channel integrations, exec approvals, cron jobs, agent sessions, and operational maintenance.
---


# OpenClaw Ops

You are an expert OpenClaw administrator. Use the scripts below to diagnose and fix issues — they contain the implementation logic. Reach for scripts first; only write manual steps when no script covers the case.

## Reference Documentation

- [cli-reference.md](docs/cli-reference.md) — Complete CLI command reference
- [troubleshooting.md](docs/troubleshooting.md) — Common issues and solutions
- [channel-setup.md](docs/channel-setup.md) — Platform-specific setup guides
- [security-guide.md](docs/security-guide.md) — Active security defense guide
- [docs.openclaw.ai](https://docs.openclaw.ai) — Official documentation

---

## Scripts

All scripts live in `scripts/` relative to this skill (typically `~/.openclaw/skills/openclaw-ops/scripts/`). Always use that full path when suggesting commands to users.

On Knox's machine, the canonical ops checkout is `/Users/knox/Developer/openclaw-ops`. Prefer those scripts over the installed skill snapshot under `~/.agents/skills/openclaw-ops`, which can lag behind. If the two differ, treat the Developer checkout as authoritative and sync/reinstall the skill snapshot as follow-up work.

| Script | When to use |
|--------|-------------|
| `heal.sh` | First thing on any health check — fixes gateway, auth mode, exec approvals, crons, and stuck sessions in one pass |
| `post-update.sh` | Run after `openclaw update` — orchestrates check-update, heal, workspace reconcile, security scan, and final health check in sequence |
| `watchdog.sh` | Continuous monitoring; run every 5 min via LaunchAgent. HTTP health check → auto-restart → escalation after 3 failures |
| `watchdog-install.sh` | Set up the watchdog as a macOS LaunchAgent (survives reboots) |
| `watchdog-uninstall.sh` | Remove the LaunchAgent |
| `check-update.sh` | After a version change — detects breaking config changes, explains them; `--fix` to auto-repair |
| `health-check.sh` | URL/process health checks for gateway-adjacent services; copy `templates/health-targets.conf.example` first |
| `session-monitor.sh` | Agent is alive but misbehaving — retry loops, hangs, auth loops, noisy failures |
| `session-search.sh` | Search session history by keyword; redacts secrets by default |
| `session-resume.sh` | Build a readable markdown resume for a single session (compaction-first, then point-of-failure) |
| `prompt-truncation-report.sh` | Report bootstrap truncation warnings from the latest session per agent. Use when users say “prompt too long,” “instructions too long,” or the bootstrap context looks incomplete. |
| `cron-optimize.sh` | Audit agent cron jobs for missing `--light-context`; `--fix` enables it and adds a default thinking level only when one is not already set. |
| `cron-error-inspector.sh` | Format erroring cron jobs from cron state, including last error, reason, consecutive count, last-run age, and a truncated payload preview. |
| `remediation-board.sh` | Human/agent repair board for surfaced ops findings. Import cron or machine incidents, track recurring bugs, hacks/workarounds, upstream watches, incident notes, hypotheses, steps tried, and verification state. |
| `agent-dirs-audit.sh` | Audit unconfigured dirs under `~/.openclaw/agents/`. Default is dry-run; `--archive` moves dormant dirs to `_archived/YYYY-MM-DD/`, `--delete-empty` removes empty dirs. |
| `backup-rotate.sh` | Rotate generic `*.bak*` files across `~/.openclaw`, grouped by the path prefix before `.bak`. Keeps the newest N per group; dry-run by default, `--apply` to delete. |
| `context-audit.sh` | Audit AGENTS.md, MEMORY.md, and SOUL*.md for file bloat. Reports path, token estimate (chars/4), and mtime, ranked largest-first above a token threshold. |
| `session-purge.sh` | Reclaim disk + cut session context bloat. Purges stale session index entries, orphan cron/subagent sessions, old `.bak` files, and orphan `.jsonl` transcripts. Dry-run by default; `--apply` to execute. |
| `workspace-auto-commit.sh` | Commit dirty OpenClaw workspace repos locally. Defaults to `~/.openclaw/workspace`; use `--workspace PATH` for an agent repo or `--all` for every `workspace*` repo. Never pushes. |
| `workspace-git-audit.sh` | Audit `~/.openclaw/workspace*` repos for git status and auto-commit cron coverage. Use `--show-cron` to print suggested cron add commands for uncovered repos; `--strict` fails on uncovered or dirty repos. |
| `daily-digest.sh` | Incident, activity, watchdog, and cost summary for the last N hours |
| `incident-manager.sh` | Sourced helper for incident lifecycle (used by session-monitor and other scripts) |
| `skill-audit.sh` | Before `clawhub install` — scan skill for secrets, injection, dangerous commands; outputs LOW/MEDIUM/HIGH risk score |
| `security-scan.sh` | Config hardening compliance check (0-100); `--fix` for auto-repair; `--drift` for file change detection; `--credentials` to scan for leaked secrets; `--include-sessions` includes bulky logs/session/runtime files normally skipped |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cathrynlavery/openclaw-ops](https://github.com/cathrynlavery/openclaw-ops) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
