---
trigger: always_on
description: These instructions apply to humans and coding agents changing `pi-herdr-agents`.
---

# Repository instructions for agents

These instructions apply to humans and coding agents changing `pi-herdr-agents`.

## What this package is

`pi-herdr-agents` (Pi Herdr Agents) is a Pi extension that launches asynchronous Pi child agents exclusively in Herdr. Ordinary runs group child panes in extension-owned `Agents` tabs by default. Writing tasks may opt into one isolated Herdr-managed Git worktree per branch. Legacy role definitions that request an external CLI fail before Herdr creates resources.

The extension is fire-and-forget: `subagent` returns an acknowledgement, and completion is delivered to the parent automatically. Never add polling guidance that tells callers to sleep, tail sessions, or repeatedly check status.

## Read these first

- [`README.md`](./README.md) — canonical installation, API, configuration, lifecycle, and agent-authoring reference
- [`docs/README.md`](docs/README.md) — map of shipped contracts, active design, ADRs, and background research
- [`CONTEXT.md`](CONTEXT.md) — workflow-domain glossary; read it before changing orchestration design
- [`docs/adr/0003-installable-role-packs.md`](docs/adr/0003-installable-role-packs.md) — installable role-pack discovery, precedence, and collision contract
- [`docs/worktree-subagents.md`](docs/worktree-subagents.md) — canonical worktree operating, review, recovery, and cleanup guide
- [`RELEASING.md`](RELEASING.md) — release checks and publishing procedure

Bundled role prompts live in [`agents/`](agents/). The native `/skill:orchestrate` public-review fan-out skill lives at [`skills/orchestrate/SKILL.md`](skills/orchestrate/SKILL.md). The `/plan` orchestration prompt lives at [`pi-extension/subagents/plan-skill.md`](pi-extension/subagents/plan-skill.md).

## Code map

- `pi-extension/subagents/index.ts` — public tools/commands, agent discovery, launch/watch lifecycle, completion delivery, worktree manifests and handoffs
- `pi-extension/subagents/herdr.ts` — Herdr CLI calls, response parsing, and ID-based Agents tab placement and capacity
- `pi-extension/subagents/terminal.ts` — terminal adapter used by the lifecycle
- `pi-extension/subagents/lifecycle.ts`, `status.ts`, `activity.ts` — process/turn state and widget projection
- `pi-extension/subagents/wake.ts`, `supervision.ts`, `supervision-config.ts` — file wake-ups, shared pane reconciliation, polling fallback, and supervision configuration
- `pi-extension/subagents/persistent-config.ts` — strict persistent-specialist cap configuration
- `pi-extension/subagents/completion.ts`, `session.ts`, `subagent-done.ts` — child completion, transcript handling, `caller_ping`, and `subagent_done`
- `CONTEXT.md` — orchestration-domain glossary
- `docs/adr/` — hard-to-reverse architectural decisions
- `docs/research/` — evidence and alternatives, never the shipped contract
- `test/test.ts` — unit tests for public subagent extension seams
- `test/package-skill.test.js` — bundled skill and package manifest contract test
- `test/integration/` — real Herdr and Pi lifecycle tests using the deterministic provider by default
- `test/bench/supervision-bench.mjs` — manual isolated-Herdr supervision transport benchmark; raw samples stay in `/tmp/issue29-bench/`

## Worktree contract

Preserve these invariants when changing worktree behavior:

1. `worktree: { branch, base? }` is opt-in per `subagent` call.
2. `cwd` selects the source repository; the child starts at the created worktree root.
3. `base` resolves to an exact commit before creation and defaults to committed `HEAD`.
4. Parent uncommitted/untracked files are not copied.
5. An ownership manifest is written before Herdr resource creation.
6. Herdr creates the workspace without stealing focus; launch targets the returned root pane explicitly.
7. Successful, failed, and help-requesting runs retain their worktree workspace.
8. Completion reports reviewable Git state; inspection failures are unknown, never guessed clean or conflict-free.
9. The extension does not push, create PRs, merge, cherry-pick, switch the parent checkout, or remove worktrees automatically. Explicit parent-owned cleanup uses cwd containment and fail-closed eligibility; branches are never deleted.
10. Ordinary non-worktree subagent behavior remains unchanged.

Read [`docs/worktree-subagents.md`](docs/worktree-subagents.md) before changing any of these semantics.

## Orchestration guidance

- Use ordinary panes for read-only scouts and reviewers.
- A single or sequential writer can work in the parent checkout; reserve unique worktree branches for independent parallel writing tasks.
- Keep overlapping or dependent writing tasks sequential unless the dependency is committed and used as the next exact base.
- Tell worktree workers whether to commit. A good default is: edit, test, commit, report the SHA, and do not push/merge/remove.
- The parent owns review, integration, publication, and cleanup.
- Do not use `subagent_resume` as if it reattached worktree ownership; v1 resumes into an ordinary pane.

## Documentation synchronization

When behavior changes, update every affected surface in the same commit:

- public tool parameters, role-pack protocol, or lifecycle → `README.md`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [giuseppecrj/pi-herdr-agents](https://github.com/giuseppecrj/pi-herdr-agents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
