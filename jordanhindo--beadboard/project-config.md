---
trigger: always_on
description: Execution-first, evidence-first, beads-driven.
---

# Agent Operating Manual (BeadBoard)

Execution-first, evidence-first, beads-driven.

## Core Rules

0. Memory lives in `bd` decision beads, not markdown files. See **Native Memory Workflow** below.
1. **Bead naming format is MANDATORY**: `beadboard-<epic>.x.x` for tasks and subtasks.
   - Example: `beadboard-05a.1` (task under epic beadboard-05a), `beadboard-05a.1.1` (subtask)
   - **WHY**: Tasks without parent-child relationships to an epic are ORPHANS and will NOT appear in the left panel navigation.
   - Always run `bd dep relate <epic-id> <task-id> --type parent-child` after creating a task bead.
2. **Never work without a bead**: Every task, bug fix, or feature must have a corresponding bead created BEFORE starting work.
   - Follow `docs/protocols/bead-prompting.md` when creating bead descriptions.
   - Ensure correct dependencies and parent epic are set.
3. Working directory: `codex/beadboard`. Run all `bd` and `npm` commands from here.
4. `bd` is the source of truth for work state. No direct writes to `.beads/issues.jsonl`.
5. **"yo" / "what's up" / any greeting** → run `bd query "status=closed" --sort closed --reverse --limit 10` and `bd ready`, then suggest the next bead.
6. Evidence before assertions: never claim fixed/passing/done without fresh command output.
7. Keep user-facing labels and UI copy simple and explicit.
8. Reuse shared code paths/components; avoid one-off logic drift across views.
9. BeadBoard is a multi-agent coordination system first. Optimize for swarm execution clarity over cosmetics.
10. Runtime UI is query-driven from `/` (`view=social|graph`). Do not reintroduce App Router page sprawl without approval.
11. Keep the active page surface minimal under `src/app`. Maintain backward-compatible redirects in `next.config.ts` when route contracts change.
12. **Close beads with evidence**: When finished, always close beads with detailed notes including commands run, files changed, and verification output.


## Craft Standards

- **Demand Elegance**: For non-trivial changes, ask "is there a more elegant way?" before presenting. Skip for simple, obvious fixes.
- **Autonomous Bug Fixes**: When given a bug report, diagnose and fix it. Point at evidence (logs, tests, diffs) — don't ask for hand-holding.
- **Learn from Corrections**: After any user correction, create or supersede a `mem-canonical` decision bead capturing the pattern so the mistake isn't repeated.

## Agent Identity (Required)

Every agent — orchestrator or worker — must have an agent bead before claiming any work.

1. **On session start**, create your agent bead:
   ```bash
   bd create --title="Agent: <role-name>" --description="<what this agent does>" --type=task --priority=0 --label="gt:agent,role:<orchestrator|ui|graph|backend|infra>"
   ```
2. **When claiming work**, always include `--assignee`:
   ```bash
   bd update <task-id> --status in_progress --assignee <your-agent-bead-id>
   ```
3. **When dispatching sub-agents**, their prompt must include:
   - Step 1: create their own `gt:agent` bead
   - Every `bd update --status in_progress` must include `--assignee <their-bead-id>`

Agent presence in the UI (liveness dots, graph node overlays) depends on assignee fields being populated.

## Agent Workflow

```bash
# 1. Read memory
bd show beadboard-116 beadboard-60a beadboard-zas   # hard rules

# 2. Find work
bd ready
bd show <id>                                         # read full spec + acceptance criteria

# 3. Create your agent bead (if not already done this session)
bd create --title="Agent: <role>" --type=task --priority=0 --label="gt:agent,role:<role>"

# 4. Claim
bd update <id> --status in_progress --assignee <your-agent-bead-id>

# 5. Implement (TDD: write failing test first, then code)

# 6. Verify
npm run typecheck && npm run lint && npm run test

# 7. Record evidence + close
bd update <id> --notes "<commands run and their output>"
bd close <id> --reason "<what was completed>"

# 8. Memory review
# If reusable lesson → create/supersede canonical memory node
# If no lesson → bd update <id> --notes "Memory review: no new reusable memory."

# 9. Sync
bd dolt pull && bd dolt push
```

**Wrong flags to avoid:**
- `bd close` does not support `--notes` — update first, then close
- `bd create` uses `--label` not `--labels`

## Native Memory Workflow (Required)

Memory source-of-truth is `bd` + Dolt history. Canonical memories are `type=decision` beads.

**Labels:** `mem-canonical, mem-hard|mem-soft, memory, memory-<domain>`

**Domain anchors:**
| Anchor | Domain |
|---|---|
| `beadboard-76p` | Architecture |
| `beadboard-nq9` | Workflow Protocol |
| `beadboard-5r1` | Agent Operations |
| `beadboard-fld` | UI/UX |
| `beadboard-8st` | Reliability & Errors |

**Key canonical memories (read at session start):**
- `beadboard-116` — [HARD] Evidence before completion claims
- `beadboard-60a` — [HARD] Dependencies model execution order
- `beadboard-zas` — [HARD] Shared logic for cross-view behavior
- `beadboard-dvp` — [SOFT] Parallelize independent work
- `beadboard-6fv` — [HARD] Triage stale-state via parity + watcher checks

**Creating a canonical memory:**
```bash
bd create --title="[MEMORY][DOMAIN][HARD|SOFT] Rule" \
  --description="Scope: ...\nOut of Scope: ...\nRule: ...\nRationale: ...\nFailure Mode: ..." \

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jordanhindo/beadboard](https://github.com/jordanhindo/beadboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
