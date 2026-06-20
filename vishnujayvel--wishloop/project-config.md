---
trigger: always_on
description: |
---


# Wishloop v2

Thin session manager for Loki Mode. Classify work, spec it via OpenSpec CLI, configure and launch Loki, monitor the session, babysit the PR, loop.

## Prerequisites

- **openspec CLI** — `openspec` (spec generation)
- **loki CLI** — `loki start` (autonomous execution, requires `--dangerously-skip-permissions`)
- **gh CLI** — `gh issue create` / `gh issue list` (bug filing and feedback loop)
- **Worktrunk** *(recommended)* — `brew install worktrunk && wt config shell install` (worktree lifecycle, merge automation). Optional — the skill falls back to manual git worktrees when not installed.

## Design Principles

| Principle | Rule |
|-----------|------|
| **P1: Configure, Don't Reimplement** | If Loki has a capability, configure it via env vars, flags, or CLAUDE.md. Never rewrite it. |
| **P2: Monitor, Fix Post-Session** | Lightweight alive/dead/stalled check during session. Fix gaps after. |
| **P3: Match Session Type to Work** | `loki start --parallel` for features. `loki run #N --pr` for single bugs. `loki quick` for review fixes. |
| **P4: Strong Proposals, Thin Wrapper** | Quality of the Loki session = quality of the proposal. Invest in enrichment. |
| **P5: Upstream When Generic** | If a Wishloop feature is generic, contribute upstream. Plan B after 2 weeks. |
| **P6: External Interface Only** | Wishloop calls ONLY Loki's public CLI. NEVER internal run.sh functions. Zero `.loki/` file coupling. |

---

## Drain Mode

Autonomous loop that processes ALL open GitHub issues until the backlog is empty. Wraps v2 Steps 1-6 in a fetch-prioritize-launch-monitor-babysit cycle.

### Invocation

```bash
bash <skill-path>/scripts/drain.sh                          # drain all open issues
bash <skill-path>/scripts/drain.sh --label bug              # drain only bugs
bash <skill-path>/scripts/drain.sh --label refactor --max-iterations 3
```

Or via the skill trigger: `wishloop --drain` / `wishloop --drain --label bug`

### How It Works

```text
FETCH (gh issue list) → STOP? (zero = exit) → PRIORITIZE → RESUME? →
  PICK → CLASSIFY → LAUNCH (loki run #N --pr) →
  MONITOR (loki status --json, 5min polls) →
  POST (docs, learnings, archival) →
  BABYSIT (CodeRabbit → loki quick → merge) →
LOOP
```

### Key Design: GitHub Issues ARE the State

No local backlog file. GitHub Issues are the single source of truth:
- Open issues = work remaining
- Closed issues = work done  
- Issue labels = priority and filtering
- PR state = current work-in-progress

### Session Resumption

On re-invoke, drain mode checks:
1. `.wishloop/state.json` — what step were we in?
2. `gh pr list --state open` — if any, resume at BABYSIT
3. `loki status --json` — if running, resume at MONITOR
4. If neither, fetch open issues and start fresh

### Monitoring (via `loki status --json`)

The drain loop polls `loki status --json` every 5 minutes to detect:
- `completed` → proceed to post-session
- `stopped` → proceed to post-session
- `running` → continue polling (with stall detection)
- `unknown` → attempt `loki resume`, then crash recovery

### Emergency Stop (`loki stop`)

If the session is stalled (iteration unchanged for 15+ minutes), drain mode runs:
```bash
loki stop     # Kill stalled session
loki resume   # Restart from last checkpoint
```

### Priority Tiers

```text
critical > bug > auto-detected > refactor > enhancement > documentation
Within same tier: smaller effort first (body length heuristic)
```

### Cumulative Summary (on exit)

```text
=== Drain Complete ===
Iterations: 3 | Duration: 47 min | Commits: 12 | Files: 8
Issues at start: 5 | Resolved: 4 | Filed: 1 | Remaining: 2
Loki sessions: 3 | PRs merged: 4 | Quick fixes: 2
```

---

## Step 1: Intake

Classify the work request into a type and route accordingly.

| Signal | Work Type | Spec? | Loki Session Type |
|--------|-----------|-------|-------------------|
| "build X from scratch", greenfield | **Greenfield** | Full OpenSpec init | `loki start --parallel` |
| "add X", "integrate Y", new feature | **Feature** | `openspec new change` | `loki start --parallel` |
| "refactor", "migrate from X to Y" | **Refactor** | `openspec new change` (MODIFIED) | `loki start --parallel` |
| "fix #N", single issue | **Bug fix (single)** | None | `loki run #N --pr` |
| "fix these bugs", bug list | **Bug batch** | Batched changes | `loki start --parallel` per wave |
| "add tests", "E2E coverage" | **Testing** | Test-focused change | `loki start --parallel` |
| "write docs", "API docs" | **Documentation** | None | `loki docs generate` (no session) |
| "design X", "architecture for Y" | **Architecture** | Proposal + design only | None |
| "research", "spike", "evaluate" | **Research** | None | None |
| "audit X", "review Y for Z" | **Audit** | None | None |
| "rethink the X", "brainstorm" | **Product Thinking** | None | None |

**Non-code work types** (Architecture, Research, Audit, Documentation, Product Thinking): Load the matching template from `templates/` and skip to output. No Loki session needed.

| Work Type | Template |
|-----------|----------|
| Architecture (HLD) | `templates/hld.md` |
| Architecture (LLD) | `templates/lld.md` |
| Research | `templates/research.md` |
| Audit | `templates/audit.md` |
| Documentation | `templates/docs.md` |

---

## Step 2: Spec + Enrich

For code work types, produce the input that Loki needs.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vishnujayvel/wishloop](https://github.com/vishnujayvel/wishloop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
