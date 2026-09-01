---
trigger: always_on
description: This file is the entry point for AI coding agents (Codex, Claude, etc.) working on this repository. `CLAUDE.md` is a
---

# Repository guide for AI agents

This file is the entry point for AI coding agents (Codex, Claude, etc.) working on this repository. `CLAUDE.md` is a
symlink to this file, so both conventions resolve to the same content.

It is loaded into every agent session — keep it short. It routes to the authoritative documents; detail belongs
there, not here. Update this file only when repository-wide agent instructions, safety rules, or documentation
routing change — never to track a moved handler, helper, module, or test.

## What this project is

`chipping-orchestrator` is a GitHub-Issue-driven workflow that watches issues on configured repos, drives them through a
label-based state machine, and spawns local CLI agents (`codex`, `claude`) in per-issue git worktrees to implement
them and open PRs. State lives entirely in GitHub (one workflow label + one pinned JSON comment per issue), so the
orchestrator process is stateless.

## Where the details live

[`docs/README.md`](docs/README.md) is the documentation landing page: it maps every page in the set, names the focused
page under each area, and explains which addresses stay stable. Open it when the area below is not obviously the right
one.

- User-facing overview: [`README.md`](README.md)
- Architecture, module ownership, process / agent / push model: [`docs/architecture.md`](docs/architecture.md), with
  the per-package inventory on the pages under [`docs/architecture/`](docs/architecture/)
- Workflow state machine (labels, per-tick flow, pinned state, stage handlers):
  [`docs/state-machine.md`](docs/state-machine.md), with the label, delivery-stage, conversation-stage, and lifecycle
  pages under [`docs/state-machine/`](docs/state-machine/)
- Agent roles, conversation contracts, command specs, session lifecycles: [`docs/workflow.md`](docs/workflow.md), with
  the role, conversation, and spec pages under [`docs/workflow/`](docs/workflow/)
- Configuration / env vars: [`docs/configuration.md`](docs/configuration.md) is the reference, with the observability
  settings and the operator runbooks under [`docs/configuration/`](docs/configuration/); basic knobs in
  [`.env.example`](.env.example), common advanced overrides in [`.env.example.advanced`](.env.example.advanced)
- Observability: [`docs/observability.md`](docs/observability.md) maps every surface, with the event-stream,
  trajectory, database, dashboard, and usage-parser pages under [`docs/observability/`](docs/observability/)
- Security checklist and operator-owned controls: [`docs/security.md`](docs/security.md)
- Development conventions and the pre-push checklist:
  [`.agents/skills/develop/SKILL.md`](.agents/skills/develop/SKILL.md)

## Repository layout

Top level only. Which module owns what is in
[`docs/architecture.md`](docs/architecture.md#top-level-layout) and the focused pages under
`docs/architecture/`, which are the single place that inventory is maintained.

- `orchestrator/` — the Python package. The root carries the version, `cli.py` (the composition point the
  `chipping-orchestrator` console script calls), and `__main__.py` (the `python -m orchestrator` form over it); every
  other module lives under a subpackage named after what it owns: `workflow/` (tick loop, label dispatch, and the
  per-label stage handlers under `workflow/stages/`), `github/`, `git/`, `agents/`, `scheduler/`, `config/`,
  `skills/`, `runtime/` (the polling process), `apps/` (the `streamlit run` targets), and `observability/` (analytics
  sink, dashboards, usage parser).
- `tests/` — pytest suite, mirroring the package layout (`tests/workflow/`, `tests/github/`, `tests/git/`, …).
  In-memory GitHub doubles live in `tests/support/github/`.
- `docs/` — architecture, state-machine, workflow, configuration, observability, and security references.
- `plans/` — human working notes, not specifications (see below).
- `analytics-db/` — operator-owned local analytics database (see below).
- `run.sh` — production launcher that auto-restarts after self-modifying merges.
- `.env.example` / `.env.example.advanced` — basic and advanced configuration templates.

## Development

Read [`.agents/skills/develop/SKILL.md`](.agents/skills/develop/SKILL.md) before changing anything under
`orchestrator/`, `tests/`, or `docs/`. It carries the commands, commit format, license headers, test placement,
comment rules, and dependency policy.

The repo targets Python 3.12+ and installs from the lockfile with [`uv`](https://github.com/astral-sh/uv):

```sh
uv sync --locked                              # creates .venv/ and installs runtime + dev deps from uv.lock
uv run pytest tests                           # run the test suite
uv run python -m orchestrator --once          # one polling tick then exit
```

Tests are the primary correctness gate. Add or update tests for any behavioral change.

## Safety and compatibility

- **`analytics-db/data/` is off limits.** It is the operator-owned Docker bind mount holding the local analytics
  Postgres volume — runtime state, not source. **Never traverse, read, modify, permission-repair, delete, or re-run
  any command against it with elevated privileges.** If a tool reports it as unreadable, that is expected: target

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chippingway/orchestrator](https://github.com/chippingway/orchestrator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-01 -->
