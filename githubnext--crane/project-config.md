---
trigger: always_on
description: Crane is an automated code-migration platform built on [GitHub Agentic Workflows](https://github.github.com/gh-aw/setup/quick-start/).
---

# Crane

Crane is an automated code-migration platform built on [GitHub Agentic Workflows](https://github.github.com/gh-aw/setup/quick-start/).

It runs planned, verified migrations from one language (or runtime) to another. Each iteration advances a living migration plan by one step, verifies that the system still works, and keeps the change only if correctness is preserved. Crane is a sibling of [autoloop](https://github.com/githubnext/autoloop) — same agentic-loop shape, specialized for migration rather than open-ended optimization.

## Architecture

```
crane/
├── AGENTS.md                           ← you are here
├── workflows/
│   ├── crane.md                        ← main crane workflow (compiled by gh-aw)
│   ├── shared/
│   │   └── reporting.md
│   └── scripts/
│       └── crane_scheduler.py          ← scheduler (see workflows/crane.md)
├── .crane/
│   └── migrations/                     ← migrations (directory-based and bare-markdown)
│       ├── stats_py_to_ts/
│       │   ├── migration.md            ← source, target, strategy, verification
│       │   └── code/                   ← evaluator, parity corpus, source/target staging
│       └── flask_to_fastapi.md
└── .github/
    ├── ISSUE_TEMPLATE/
    │   └── crane-migration.md          ← issue template for creating migrations
    └── workflows/                      ← compiled workflow (*.lock.yml, generated)
```

## Key Concepts

### Migrations

A **migration** defines a single port from a source language/runtime to a target. Each migration has:

- **Source**: language, version, runtime, and paths being migrated *from*
- **Target**: language(s), runtime, and paths being migrated *to* (multiple target languages allowed — e.g. TypeScript with a Go core for hot paths)
- **Strategy**: `in-place`, `greenfield`, or `auto`
- **Verification**: a command that outputs a JSON health score combining correctness with progress
- **Completion Gate**: deterministic PR-head CI or check-run evidence required before Crane may mark the migration complete

Migrations can be:

- **Directory-based** (`.crane/migrations/<name>/migration.md`): for migrations with a parity corpus or custom evaluator. Code lives in `code/`. Preferred when verification needs supporting fixtures.
- **Bare-markdown** (`.crane/migrations/<name>.md`): for migrations that modify existing repo code where verification is an existing repo command (e.g. `make test`).
- **Issue-based** (GitHub issue with `crane-migration` label): for migrations created and steered directly from a GitHub issue. The issue body uses the same format as `migration.md`. The issue itself becomes the interface for monitoring and steering.

### The Plan

Crane treats planning as a first-class step. Every migration has a **living plan** stored in its state file on the `memory/crane` branch, with these sections:

- **🗺️ Inventory** — modules in the source, their dependencies, their consumers, their test coverage, their risk
- **🧭 Strategy & Rationale** — `in-place` vs `greenfield` and why
- **🪜 Milestones** — ordered list of units to migrate. Each milestone has a name, scope, status (`todo` / `in-progress` / `done` / `blocked`), and acceptance criteria (what verification it needs to pass to be marked done)
- **🎯 Current Focus** — the one milestone the next iteration will work on
- **📚 Lessons Learned** — what worked, what didn't, accumulated across iterations
- **🚧 Blockers & Foreclosed Approaches** — dead ends with the reasons they failed
- **🔭 Future Work** — ideas surfaced but not yet promoted to milestones

The plan is generated on the **first iteration** (inventory + strategy + initial milestone list) and revised on every subsequent iteration. Humans can edit any section directly on the `memory/crane` branch to steer the migration.

### Workflow

The workflow (`workflows/crane.md`) is compiled by `gh aw compile` into `.github/workflows/crane.lock.yml`. It:

1. Runs on a schedule (every 6h by default)
2. Checks which migrations are due (reading state files from repo-memory)
3. Selects the most-overdue migration
4. Runs **one iteration**:
   - **First iteration**: inventory the source, pick a strategy (if `auto`), write the initial plan, commit the plan
   - **Subsequent iterations**: read the plan, pick the next milestone, implement it, verify, accept or reject
5. Commits accepted changes to `crane/<migration-name>`
6. Updates the state file with iteration history, plan changes, and the new health score
7. If the migration has a `target-metric` and the health score reaches it (typically `1.0` for "fully migrated and verified"), records a completion candidate
8. Marks the migration complete only after the current Crane PR head has deterministic terminal-success checks

Branch freshness is handled by the iteration loop: each iteration's branch-setup step fast-forwards or merges `origin/main` into the `crane/*` branch as needed.

### Verification (the Health Score)

Each migration defines a verification command that prints JSON containing `migration_score` — a number in `[0.0, 1.0]` where `1.0` means the migration has reached its target and is ready for the deterministic completion gate.

The recommended convention is:

```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [githubnext/crane](https://github.com/githubnext/crane) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-22 -->
