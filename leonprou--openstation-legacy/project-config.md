---
trigger: always_on
description: Task management system for coding AI agents. Convention-first —
---

# Open Station

Task management system for coding AI agents. Convention-first —
markdown specs + skills, minimal dependencies. Adding packages
or modules to existing components is fine; stay minimal.
Do not update CHANGELOG.md unless creating a new release.

## Vault Structure

Artifacts (the source of truth) live under `artifacts/<kind>/` at
the project root — this matches the artifacts-os layout that
Open Station builds on top of. Framework plumbing and runtime
state live in `.openstation/` (hidden). Project settings live in
the single-file `openstation.yaml` at the project root.

```
artifacts/                   — Artifact source-of-truth (artifacts-os layout)
  tasks/                     —   Task files (canonical location, never move)
  agents/                    —   Agent specs (canonical location)
  research/                  —   Research outputs
  specs/                     —   Specifications & designs
  notes/                     —   Planning notes (roadmap, release plans)
  alerts/                    —   Alert artifact files (triggers, activity logs)
  logs/                      —   Run logs
  artifacts.yaml             —   artifacts-os marker / per-kind config

openstation.yaml             — Project settings (single file at root)

.openstation/                — Framework plumbing + runtime (hidden)
  docs/                      —   Project documentation (lifecycle, task spec)
  skills/                    —   Agent skills (operational knowledge, not user-invocable)
  commands/                  —   User-invocable slash commands
  templates/                 —   Templates (agent specs, settings)
  hooks/                     —   Hook bundles (per-slug dirs)
  state.db                   —   Runtime state (runs, counters; rebuildable)
  events/                    —   Event log (YYYY-MM-DD.jsonl)
```

Note: agent discovery for Codex/Claude is provided by
`.Codex/agents/` (or `.claude/agents/`) symlinks that point at
`artifacts/agents/`.

## How Docs Connect

```
                        ┌──────────┐
                        │ AGENTS.md│
                        └────┬─────┘
                             │
                  references │
             ┌───────────────┤
             ▼               ▼
      ┌────────────┐  ┌───────────┐
      │lifecycle.md │◄─┤task.spec.md│
      └──┬──────┬──┘  └───────────┘
         │      │
         │      └───────────┐
         ▼                  ▼
┌─────────────────────────────┐
│  skills/                    │
│  openstation-execute/       │──► lifecycle.md
└────────────────┬────────────┘    task.spec.md
                 │                 /openstation.create
      skills:    │                 /openstation.done
      execute    │
         ┌───────┴───────┐
         ▼               ▼
  ┌──────────┐    ┌──────────┐
  │researcher│    │  author  │
  └──────────┘    └──────────┘

┌─────────────────────────────────────────┐
│  commands/                               │
│  openstation.create.md  ──► lifecycle.md │
│  openstation.done.md    ──► lifecycle.md │
│  openstation.update.md  ──► lifecycle.md │
│  openstation.list.md                     │
│  openstation.verify.md                   │
└─────────────────────────────────────────┘
```

- **task.spec.md** — the shape (schema, naming, format)
- **lifecycle.md** — the state machine (transitions, ownership, artifacts)
- **storage-query-layer.md** — the storage model (canonical paths, frontmatter associations, queries)
- **execute skill** — the agent playbook (discovery, execution, completion)

## Task Structure

Each task is a single markdown file (`NNNN-slug.md`) stored
in `artifacts/tasks/`. See `docs/storage-query-layer.md` for
the full storage and query model.

## Spec Format

All specs use YAML frontmatter with a `kind` field (e.g., `task`,
`agent`, `spec`, `doc`) followed by markdown content. Every spec
must have at minimum `kind` and `name` fields.

## Creating a New Task

Use `openstation create "<description>"` or `/openstation.create`
to create tasks — both handle ID assignment and file creation.

For manual creation, see `docs/task.spec.md` for the format.

## CLI

The `openstation` CLI provides scriptable access to the vault.
See `docs/cli.md` for the full reference (flags, exit codes,
resolution rules).

```bash
openstation list [--status <s>] [--assignee <name>]
openstation show <task>
openstation create "<description>" [--assignee <a>] [--owner <o>] [--status <s>] [--parent <p>]
openstation status <task> <new-status>
openstation run <agent> [-i] [--worktree] [--dry-run]
openstation run --task <id> [-i] [-d] [--context-only] [--worktree] [--dry-run]
openstation run --task <id> --verify [-i] [--worktree]
openstation agents [list] [--json | --quiet]
openstation agents show <name> [--json | --editor]
openstation alerts list [--type <t>] [--status <s>]
openstation alerts show <name>
openstation heartbeat
```

## Running an Agent

```bash
openstation run researcher -i              # interactive session
openstation run --task 0042 -i             # interactive task session
openstation run --task 0042                # autonomous (foreground)
openstation run --task 0042 -d             # detached (background)
openstation run --task 0042 --context-only  # context loaded, no auto-execute

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [leonprou/openstation-legacy](https://github.com/leonprou/openstation-legacy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
