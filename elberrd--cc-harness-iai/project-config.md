---
trigger: always_on
description: Minimal Claude Code template with a task-driven workflow:
---

# Project Instructions

Minimal Claude Code template with a task-driven workflow:

1. **Write the PRD** in `ai-docs/PRD.md` (replace all `{{placeholders}}`).
2. (Optional) **Create the design system** with `/design create` — generates `/DESIGN.md` at the repo root.
3. **Generate the task list** with `/create-tasks` — produces `ai-docs/todos/task-master.md` from the PRD.
4. **Execute the next task** with `/dev` — prepares, implements, and audits against `DESIGN.md`.

## Available commands

| Command | What it does |
|---|---|
| `/create-tasks` | Reads `ai-docs/PRD.md` and generates `ai-docs/todos/task-master.md` with sequential tasks and explicit dependencies. |
| `/dev [description \| --task <selector>] [--test] [--worktree] [--no-branch] [--list] [--dry-run] [--quick] [--ship]` | Selects, prepares, and implements a task. Selection: no args → next pending task in `task-master.md`; `--task <id>` → specific task; `--task 5-10` → range; `--task 5,7,9` → batch (sequential); free-form description → ad-hoc task via `ad-hoc-task-creator` (branch `task/adhoc-<slug>`, NOT in `task-master.md`). Modifiers: `--worktree` creates a git worktree and switches into it; `--no-branch` legacy/hotfix mode; `--list` prints pending/blocked tasks read-only and exits; `--dry-run` writes the task file but skips implementation (single task only); `--quick` skips specialist research and the design-system audit; `--test` runs Playwright E2E at the end; `--ship` auto-pushes + opens PR + queues auto-merge (squash + delete branch) once Steps 3/4/4.5 pass. Without `--ship`, the run ends asking the user whether to merge each PR. The design-system audit runs by default when frontend files were touched. |
| `/design [create\|lint\|check\|export\|spec]` | Manages `/DESIGN.md` (Google Labs alpha spec). `create` invokes the curator to compose the file; `check` audits components against the tokens. |
| `/learning [description]` | Records a lesson learned in `ai-docs/lessons.md` (date, context, root cause, how to avoid, tags). Use after any avoidable mistake. |
| `/manual-verify [request]` | Runs a free-form verification you describe — browser checks via Playwright MCP, CLI checks, etc. — and reports anything that needs human action. |

## Layout

```
README.md                   # How to use this template
CLAUDE.md                   # Project instructions (this file)
DESIGN.md                   # Visual tokens (created by /design create — Google Labs alpha spec)

ai-docs/
├── PRD.md                  # Requirements doc (you fill it in)
├── lessons.md              # Lessons learned (updated by /learning)
├── tools.yaml              # Catalog of tools the agents may use
├── todos/
│   └── task-master.md      # Generated task list (created by /create-tasks)
└── actual-todo/
    └── NN-name.md          # Task currently in flight (created by /dev)

.claude/
├── agents/
│   ├── task-master-generator.md     # generates task-master.md from the PRD
│   ├── task-sequencer.md            # prepares the next PRD task file
│   ├── ad-hoc-task-creator.md       # prepares an off-roadmap ad-hoc task file
│   ├── design-system-curator.md     # creates/updates DESIGN.md
│   ├── design-system-checker.md     # audits components against DESIGN.md (auto in /dev)
│   └── quality-checklist-verifier.md # auto-verifies each task's quality checklist (auto in /dev)
├── commands/
│   ├── create-tasks.md              # /create-tasks
│   ├── dev.md                       # /dev
│   ├── design.md                    # /design
│   ├── learning.md                  # /learning
│   └── manual-verify.md             # /manual-verify
├── hooks/
│   └── block-npm-npx.sh             # PreToolUse hook — redirects npm/npx → pnpm (optional)
├── skills/                          # bundled Skills (stack-specific — keep what you use)
└── settings.json                    # agent-teams env + hook wiring
```

## Invariant rules

### Tasks
- Before generating tasks, `task-master-generator` analyzes the existing codebase to **avoid duplicating** work that's already implemented.
- Tasks have explicit `dependencies` — a task can only start when ALL its dependencies are `done`.
- **Each task is implemented on its own `task/<NN>-<slug>` branch** (created automatically by `/dev`). Branch existence is the **claim**: a tab/session that finds an existing `task/<NN>-*` branch (locally or on remote) skips that task and picks the next one.
- Before the final commit, `/dev` runs `git fetch && git rebase <base-branch>` to pull in `task-master.md` updates from already-merged parallel branches.
- Each task only modifies its OWN row in `task-master.md` (status: `done`). Dependency state (`blocked` ↔ `pending`) is computed lazily by `task-sequencer` — never written eagerly. This keeps parallel PRs conflict-free.
- When finishing a task via `/dev`, update its status in `task-master.md` to `done`.
- **Working tree limpo antes de claim.** `task-sequencer` recusa começar uma task nova se houver mudanças não commitadas. Mudanças de infra (`.claude/**`, `CLAUDE.md`, `ai-docs/PRD.md`, `ai-docs/tools.yaml`, etc.) devem ser commitadas em `<BASE_BRANCH>` como `chore(workflow): ...` antes de iniciar a próxima task. Step 0.4.0 do `task-sequencer` aborta com classificação META/TASK.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [elberrd/cc-harness-iai](https://github.com/elberrd/cc-harness-iai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-26 -->
