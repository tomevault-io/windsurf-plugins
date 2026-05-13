---
trigger: always_on
description: Zapat is an autonomous dev pipeline framework powered by Claude Code. It triages GitHub issues, implements features, runs tests, reviews code, and auto-merges -- all triggered by a single label.
---

# Zapat

Zapat is an autonomous dev pipeline framework powered by Claude Code. It triages GitHub issues, implements features, runs tests, reviews code, and auto-merges -- all triggered by a single label.

## Getting Started

If `.env` does not exist, run `/zapat` (from the repo) or `/zapat:setup` (from the plugin) to configure Zapat for your project.

If already configured, run `/pipeline-check` or `/zapat:pipeline-check` to verify everything is healthy.

## Quick Reference

### Labels

| Label | Description |
|-------|-------------|
| `agent` | Let the pipeline handle this (works on issues and PRs) |
| `agent-work` | Skip triage, implement immediately |
| `agent-research` | Research and analyze, don't code |
| `agent-write-tests` | Write tests for the specified code |
| `agent-plan` | Proposed work, pending human approval (not auto-implemented) |
| `agent-phase-2` | Phase 2 work, awaiting Phase 1 completion |
| `agent-phase-3` | Phase 3 work, awaiting Phase 2 completion |
| `hold` | Block auto-merge on this PR |
| `human-only` | Pipeline should not touch this |
| `agent-full-review` | Force full team review regardless of complexity |
| `codex` | Process with OpenAI Codex |
| `claude` | Process with Claude Code |

Status labels are managed automatically by the pipeline:

| Label | Meaning |
|-------|---------|
| `zapat-triaging` | Triage in progress |
| `zapat-implementing` | Implementation in progress |
| `zapat-review` | Code review pending |
| `zapat-testing` | Tests running |
| `zapat-researching` | Research in progress |
| `zapat-rework` | Addressing review feedback |
| `zapat-visual` | Visual verification in progress |
| `zapat-ci-fix` | CI auto-fix in progress |
| `needs-rebase` | Auto-rebase failed due to conflicts (manual resolution needed) |

Classification labels (applied during triage):

| Label | Description |
|-------|-------------|
| `feature` | New feature |
| `bug` | Bug fix |
| `tech-debt` | Technical debt |
| `security` | Security issue |
| `research` | Research task |

Priority labels (applied during triage):

| Label | Description |
|-------|-------------|
| `P0-critical` | Critical priority |
| `P1-high` | High priority |
| `P2-medium` | Medium priority |
| `P3-low` | Low priority |

### CLI Commands

| Command | Description |
|---------|-------------|
| `bin/zapat status` | Pipeline overview (active sessions, recent jobs, success rate) |
| `bin/zapat health` | Run health checks on all pipeline components |
| `bin/zapat health --auto-fix` | Auto-repair common issues (orphaned worktrees, stale slots) |
| `bin/zapat metrics query --days 7` | Query job metrics for the last N days |
| `bin/zapat risk REPO PR_NUM` | Classify risk level of a pull request |
| `bin/zapat dashboard` | Launch the Next.js monitoring dashboard |
| `bin/zapat program <issue>` | Track multi-issue progress, dependencies, and ETA |
| `bin/zapat logs rotate` | Rotate and compress old log files |

### Skills (project-scoped)

| Skill | Description |
|-------|-------------|
| `/zapat` | Configure Zapat for your project (interactive wizard) |
| `/add-repo` | Add a new repository to monitor |
| `/pipeline-check` | Quick health check with plain-language results |

### Skills (plugin — available globally after `claude plugin install`)

| Skill | Description |
|-------|-------------|
| `/zapat:setup` | Configure Zapat for your project (interactive wizard) |
| `/zapat:add-repo` | Add a new repository to monitor |
| `/zapat:pipeline-check` | Quick health check with plain-language results |

## Architecture

> **Detailed architecture reference:** [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md) is the single source of truth for system design — pipeline flow, state machine, label protocol, concurrency, prompt architecture, risk scoring, and extension points.

```
GitHub Issue (labeled) --> Poller (every 2 min) --> Trigger Script --> Claude Code Agent Team --> PR --> Review Agent Team --> Auto-Merge Gate
```

**Flow:**
1. The cron-based poller (`bin/poll-github.sh`) scans configured repos for issues/PRs with pipeline labels.
2. When found, it dispatches the appropriate trigger script (`triggers/on-new-issue.sh`, `triggers/on-work-issue.sh`, etc.) in a background tmux session.
3. The trigger script fetches `origin/main` and creates an isolated worktree under `~/.zapat/worktrees/` before launching agents, ensuring they always see the latest code.
4. All job types (triage, review, research, implementation, rework) run in isolated git worktrees — never touching the user's main checkout. Implementation worktrees create PRs; read-only worktrees are cleaned up after the session.
5. The auto-merge gate evaluates risk (low/medium/high) and merges if all checks pass.
6. When a PR merges and `main` moves forward, the auto-rebase system detects stale `agent/*` PRs and rebases them automatically. On conflict, it adds the `needs-rebase` label and posts details.

**Key directories:**

| Directory | Purpose |
|-----------|---------|
| `bin/` | Pipeline CLI and core scripts (poller, startup, notifications) |
| `triggers/` | Event handlers that launch agent teams |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zapat-ai/zapat](https://github.com/zapat-ai/zapat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
