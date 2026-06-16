---
trigger: always_on
description: `AGENTS.md` is a symlink to this file. Keep project-agent instructions here so both entry points stay identical.
---

## Source of Truth for Project State

`AGENTS.md` is a symlink to this file. Keep project-agent instructions here so both entry points stay identical.

**Nested `AGENTS.md` files carry local contracts.** Some directories have their own `AGENTS.md` with rules local to that subtree (e.g. `apps/AGENTS.md`). Before editing any file, read the `AGENTS.md` in its directory if one exists — those rules add to, and never override, this root file.

**CLAUDE.md does not track in-progress work or completion status.** It goes stale immediately and will mislead future sessions.

- **What shipped and why** → `git log --oneline -20` and `GOTCHAS.md` for non-obvious discoveries
- **What's currently in flight** → `git status`, open PRs, and issue pipeline labels
- **Product direction** → `NORTH_STAR.md`
- **App-framework + marketplace plan** → `docs/prm/app-framework-marketplace.md`
- **Host UI kit plan** → `docs/prm/host-ui-kit.md`
- **File Explorer overhaul PRD** → `docs/prm/file-explorer-overhaul.md`
- **Operational sprint graph** → `.stint/` (`stint status`, `stint next`, `stint sprint show <id>`)
- **Implementation backlog** → GitHub issues, used as work tickets only

Before reporting anything as "done" or "missing", verify against `git log`.

## Website / Domain

The product website is **`plexiapp.com`**. All docs links use `https://plexiapp.com/docs/...`. Never write `plexiapp.dev` or `plexi.app` — both are wrong.

## Planning Source

Plexi no longer uses GitHub Project board #7, `NEXT.md`, or generated dispatch snapshots to decide what happens next. Do not query, update, or trust the Project board for planning.

For app-framework, packaging, marketplace, MCPUI, WASM/WASI, and Bevy work, read `docs/prm/app-framework-marketplace.md` first. That PRM is the local source of truth and resolves conflicts with older roadmap fragments.

For host-level UI chrome work (modals, overlays, palettes, rows, text fields, buttons, hint bars, and overlay focus handling), read `docs/prm/host-ui-kit.md` first. That PRM is the local source of truth for the host UI kit sequence and points to the ordered implementation issues.

For File Explorer overhaul work, read `docs/prm/file-explorer-overhaul.md` after `docs/prm/host-ui-kit.md`. The File Explorer PRD is intentionally queued behind the Host UI Kit rework; do not rebuild File Explorer-specific row, table, modal, button, text field, or hint primitives that belong in the shared host UI kit.

When a PRD has a `Progress` table, update the relevant row in the same change that finishes or supersedes an issue. Do not make future agents infer PRD completion only from closed GitHub issues.

GitHub issues are implementation tickets. To choose the next dispatch, match open issues against the first unfinished milestone in the PRM, skip anything blocked or in progress, then choose parallel lanes whose `area:*` labels do not overlap. If the PRM calls for work that has no issue yet, create or triage the issue before dispatching.

Sprint sequencing and task blockers live in `.stint/`. Use the PRM for product direction, then use `stint next` for the next claimable task. Keep `gh_issue` and `blocked_by` frontmatter current when a task is linked to GitHub or blocked by another task. (`blocked_by` is the single unified, polymorphic field — bare int = local task, `@N` = local issue, `owner/repo@N` = external issue, quoted = free-text note. The old `blocked_by_gh` split is retired.)

**The active ready pool is v1-only, enforced by task status — not a folder or a gate.** Every v1 task is `status: todo` (or `in-progress`/`done`); the entire v2 phase (81 open tasks, sprints `s15`–`s30`) lives in the same board as `status: backlog` — the icebox. `stint next` and the bottleneck calc ignore `backlog` by default, so the ready pool cues v1 in order and the bottleneck points at real v1 chains instead of the "ship v1" tautology. There is no `v2-after-v1` gate and no `v2-backlog/` folder; the `backlog`↔`todo` distinction replaces both. New ideas land as `backlog` (the `stint add` default) and become claimable only when a human runs `stint ready <id>` / `stint ready --tag <tag>`. When v1 ships (task `0030`, itself held in `backlog` so it never shows as a bottleneck), promote v2 with `stint ready --tag v2`. `blocked_by` is reserved for true artifact dependencies — express phase/ordering preference with `backlog` status and sprint sequence, not blockers.

## Stint Time Tracking

Every stint task must record actual timing when work starts and when it completes. This is required so estimates can be compared against real elapsed implementation time.

- When implementation begins, set the linked task `status` to `in-progress` and write `started_at: "<UTC ISO-8601 timestamp>"` if it is not already set.
- Use `stint start <task-id>` to do this; use `--started-at` only for historical backfill.
- When implementation is complete, run `stint done <task-id>` so `completed_at`, elapsed `actual`, and `done` status are recorded together.
- Use `stint done <task-id> --actual <duration>` only when overriding the computed/prompted actual time; use `--completed-at` only for historical backfill.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ianjamesburke/PLEXI](https://github.com/ianjamesburke/PLEXI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
