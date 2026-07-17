---
trigger: always_on
description: Registry-driven, multi-project agent infrastructure. Runs nightly via GitHub Actions, Supabase is the source of truth. Reviews 6+ codebases automatically for security, bugs, UX, content, polish, and performance issues.
---

# Infrastructure Project Context

## What This Is

Registry-driven, multi-project agent infrastructure. Runs nightly via GitHub Actions, Supabase is the source of truth. Reviews 6+ codebases automatically for security, bugs, UX, content, polish, and performance issues.

## Architecture

```
.github/workflows/
└── nightly-review.yml          # 6-job pipeline (runs 2am CST Sun/Mon/Wed/Fri)

agents/
├── registry.json               # Project + agent config (22 agents, automation profiles, schedules)
├── config/
│   └── credentials.json        # Token/key registry with expiry dates
├── orchestrator/
│   └── decide-roster.md        # AI orchestrator prompt (builds nightly roster)
├── reviews/
│   └── {project}/              # Per-project review prompts (7 themes)
│       ├── security-review.md
│       ├── ux-layout-review.md
│       ├── bug-hunt-review.md
│       ├── content-value-review.md
│       ├── polish-brand-review.md
│       ├── performance-review.md
│       └── aso-retention-review.md
├── ops/
│   └── weekly-ops-combined.md  # Sunday ops + evaluator (combined)
├── workers/
│   └── work-loop-manager.sh    # Executes approved work items
└── lib/
    ├── collect-orchestrator-signals.ts
    ├── sync-to-supabase.ts
    ├── build-agent-context.ts     # Persistent memory context injection
    └── git-activity-scanner.sh

packages/agent-learning/
└── src/
    ├── scoring/signal-score.ts
    └── learning/similarity.ts
```

## Nightly Pipeline (6 Jobs, 4x/week: Sun/Mon/Wed/Fri)

1. **setup** — Determines today's theme + project scope from day-of-week
2. **clone-projects** — Clones relevant repos from registry
3. **orchestrator** — Commit gate checks git_activity; if all quiet, skips AI call and produces empty roster. Otherwise AI builds roster.json (cap: 5 entries, falls back to static if it fails)
4. **reviews** — Runs `claude -p` for each roster entry sequentially
5. **sync** — Writes findings to Supabase `work_items` + `agent_runs_v2`
6. **workers** — Executes approved work items via `claude -p`
7. **reconcile** — Stale item sweep + combined daily ops agent (credentials + health + brief in 1 call)

## Weekly Schedule (4x/week as of 2026-04-13)

| Day | Theme | Scope |
|-----|-------|------|
| Mon | security-review | core |
| Wed | bug-hunt-review | core |
| Fri | security-review + polish (odd weeks) | core |
| Sun | weekly-ops-combined + business-synthesis + portfolio-audit (1st Sun/mo) | all |

Tue/Thu/Sat: no pipeline runs. Specialist agents (competitive-intel, content-writer, creative-provocateur, marketing-analyst) moved to on-demand only.

## Automation Profiles

- **Core**: sidelineiq, airtip, dosie, glossy-sports, mainline-apps, mainline-dashboard
- **Scaffolded**: gt-ops, menu-autopilot, the-immortal-snail, gt-website
- **Ops-only**: infrastructure

## Agent Budgets (Updated 2026-04-13)

Target: ~$100-140/month (down from ~$450). Key savings: 4x/week schedule, commit gate skips quiet projects, specialists on-demand, consolidated ops agents.

## Key Tables (Supabase)

| Table | Purpose |
|-------|---------|
| `agents` | Roster: id, name, tier (1-4), status, budget_monthly |
| `agent_runs_v2` | Run history: agent_id, project, status, cost, tokens |
| `work_items` | All findings: status flow discovered→triaged→approved→in_progress→review→done |
| `work_item_events` | Audit trail |
| `agent_budget_summary` | View: per-agent monthly cost vs budget |

## Secrets (GitHub Actions)

- `ANTHROPIC_API_KEY` — Powers all `claude -p` calls. **If credits run out, entire pipeline silently fails.** Set auto-reload at console.anthropic.com.
- `GH_PAT` — Clones private repos
- `SUPABASE_URL` / `SUPABASE_SERVICE_ROLE_KEY` — Reads/writes findings
- `COMMAND_CENTER_URL` / `COMMAND_CENTER_API_KEY` — Posts to dashboard

## Known Gotchas

- **`claude -p` in while-read loops**: Must use fd 3 (`read -u 3`, `done 3<<<`) and `< /dev/null` to prevent stdin consumption. See skill: `claude-p-while-read-stdin-consumption`.
- **Static fallback**: If orchestrator AI fails, workflow falls back to naive static roster. Check `"source": "static-fallback"` in roster JSON.
- **Worker directory mismatch**: Workers look for projects at `projects/{name}` — monorepo subdirectories must be extracted correctly in clone step.
- **Budget view uses current month**: `agent_budget_summary` resets monthly. Runs from prior months don't count against current budget. View now includes `effective_budget` (base + overrides) and `budget_enforcement_mode` (observe/warn/enforce). All agents default to `observe` while calibrating.
- **Entity grouping**: `entities` and `project_entities` tables map projects to GT/Mainline/Personal. `entity_budget_summary` view rolls up costs per entity per month.

## Recent Changes


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [clayparrish-cyber/infrastructure](https://github.com/clayparrish-cyber/infrastructure) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-17 -->
