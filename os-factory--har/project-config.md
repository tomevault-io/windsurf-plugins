---
trigger: always_on
description: Guide for coding agents working on **this repository** (`@osfactory/har` — the CLI and MCP control plane).
---

# HAR — Agent Development Guide

Guide for coding agents working on **this repository** (`@osfactory/har` — the CLI and MCP control plane).

For setup, testing fixtures, and PR workflow, see [CONTRIBUTING.md](./CONTRIBUTING.md).

This repo **dogfoods HAR** — `.har/` at the repo root defines how coding agents validate changes here.

<!-- har:agent-environment:start -->
## HAR / agent environment

The harness is **how you run this project**, not only how you verify it. This
root harness is the **CLI profile** (no runtime server). Launch a slot to get an
isolated worktree with toolchain paths; never hand-roll setup. To see Mission
Control or the docs site live, launch `control/.har/` or `docs/.har/` instead.

If a harness command fails, fix the harness (or report the failure) — do not quietly
fall back to ad-hoc commands.

### Before making changes

1. On the **main checkout**, switch to the intended base (usually `main`) — launch
   creates a worktree from that HEAD.
2. **Launch first** — MCP `har_launch_environment` / `har env launch 1`. Use the
   returned **work dir** for ALL edits (never the main checkout).
   **Bind tracker work** when the task names a durable issue or ticket (GitHub,
   Linear, etc.): pass a short repo-scoped `--work-id` / `workUnitId` (e.g.
   `widget-123`), `--work-source` / `source`, `--work-url` / `sourceUrl`, and
   `--work-title` / `title` when known. Skip binding for ad-hoc work with no
   tracker identity.
3. Read [`.har/README.md`](.har/README.md), [`.har/stages.json`](.har/stages.json), then
   [`.har/CLAUDE.agent.md`](.har/CLAUDE.agent.md) (slot URLs / definition of done).
4. Hot-reload usually applies; if not, `./.har/agent-cli.sh <id> restart` (no-op on
   cli/ios profiles without managed processes).

**Occupied slots always block.** Run `complete` / `teardown`, then `launch`. Resume
failed/starting launches with `--resume` / `recover`. Prefer a free slot (2+) over
sharing slot 1 across unrelated chats. Check `har_get_status` / `har env status` first.
Commit early — teardown keeps the branch, not uncommitted work.

### After making changes

Prefer MCP → CLI → shell. Quick verify for the loop; **full verify before done**.

- MCP: `har_run_verification` / `full: true`; finish with `har_complete_environment`
  (propose; wait for approval) or `har_teardown_environment`
- CLI: `har env verify 1`, `har env verify 1 --full`, `complete 1`, `teardown 1`
- Shell: `./.har/verify.sh 1`, `./.har/verify.sh 1 --full`, `./.har/teardown.sh 1`

Commit in the session worktree. Run JSON stays in the main checkout `.har/runs/`.

### Definition of done

- Full verify passes; edits only in the session worktree; tests cover new behavior;
  changes committed; show preview URLs; then **session handoff** (below).

### Session handoff (required)

After full verify and commit, stop. Include summary, session branch
(`.har/slots/agent-<id>.json`), and preview URLs. Wait — never autonomously
complete, teardown, push, or open a PR. **Default:** when `gh`/GitHub MCP is available,
recommend **Complete + open a PR** (still needs approval). Alternatives: **Complete only**,
or **Something else**. Without PR tooling, recommend **Complete only** and give the
session branch for a manual push.

### Commit gate

Full verify records a tree hash under `.har/validations/`. With `har hooks install`,
commits must match a passing full verify. Re-verify after any edit; `git add -A`.
Do not bypass (`--no-verify`, `HAR_SKIP_GATE=1`).

### Cursor IDE

If `.cursor/rules/har-workflow.mdc` exists, the same harness workflow is injected into
every Cursor agent session automatically. Run `har env init` or `har env maintain` to
create or refresh it.
<!-- har:agent-environment:end -->

## Harnesses in this repo

This is a monorepo with **three harnesses** — pick the one that owns the files you are changing:

| Path | Profile | Runs | Use when changing | Docs |
|------|---------|------|-------------------|------|
| `.har/` | cli | `@osfactory/har` (typecheck, build, unit tests, lint) | `src/`, `packages/`, `tests/` | [.har/README.md](.har/README.md) |
| `control/.har/` | default | Mission Control (Next.js + SQLite, browser-e2e) | `control/` | [control/.har/README.md](control/.har/README.md) |
| `docs/.har/` | default | Docs / marketing site (Astro + Playwright screenshots) | `docs/` | [docs/.har/README.md](docs/.har/README.md) |

Run harness commands from the directory that owns the harness (e.g. `cd docs && har env launch 1`). See [control/AGENTS.md](control/AGENTS.md) and [docs/AGENTS.md](docs/AGENTS.md) for project guides.

**The harness is how you run each project** — to see Mission Control or the docs site live (manual testing, browser, screenshots), launch the matching slot; never hand-roll docker/dev-server startup. If a harness command fails, fix the harness or report it — don't silently fall back to ad-hoc commands.

Docs UI work: use `docs/.har/` so full verify produces before/after screenshots under `docs/.har/artifacts/browser-e2e/screenshots/`. The root CLI harness may still run docs contract checks (`drift` / build) when changing product surfaces that the docs describe — that does not replace launching the docs harness for landing-page or Starlight UI changes.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [os-factory/har](https://github.com/os-factory/har) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-23 -->
