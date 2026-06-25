---
trigger: always_on
description: Agent-ergonomic task/backlog CLI in the `*-axi` family, built on `axi-sdk-js` and mirroring `gh-axi`.
---

# tasks-axi — agent notes

Agent-ergonomic task/backlog CLI in the `*-axi` family, built on `axi-sdk-js` and mirroring `gh-axi`.
P1 ships only the markdown backend behind a `Store` seam; sqlite (P2) and remote trackers (P3) are deferred.

## Architecture

The CLI layer never knows which backend is active — it only talks to the `Store` interface.

- `src/cli.ts` — `runAxiCli` wiring: `DESCRIPTION`, `TOP_HELP`, the verb→handler map (with aliases create/view/edit/delete/close), the optional `task` noun prefix, and the global `--backend` / `--file` flags (stripped before handlers, parsed for `resolveContext`).
- `src/context.ts` — `resolveTasksContext` builds the backend `Store` + `ResolvedConfig`; every command receives this `TasksContext`.
- `src/store.ts` — the `Store` interface and `Capabilities`. Core contract: `create/get/update/remove/list/transition/addDep/removeDep`. `prune`/`render` are optional and capability-gated.
- `src/model.ts` — the `Task` data model (report §5).
- `src/derive.ts` — `blocked` / `ready` are derived in the CLI from `list` + the dep graph, never a Store method, so every backend gets them for free.
- `src/backends/markdown*.ts` — the only P1 backend.
- `src/commands/*` — one file per verb group; `src/view.ts` owns the TOON projection.
- Shared helpers copied from the family: `args.ts`, `body.ts`, `format.ts`, `fields.ts`, `toon.ts`, `suggestions.ts`, `skill.ts`.

## Markdown grammar invariants (the hard part — do not regress)

`src/backends/markdown-grammar.ts` is pure parse/render with no I/O; `markdown.ts` adds the lock + atomic write.

- **Byte-exact round-trip (D1).** `render(parse(src)) === src` on any file nobody has mutated. Each entry keeps its exact original `raw` lines and is emitted verbatim unless `dirty`. A mutated task is re-rendered from its structured fields; untouched entries stay byte-exact. `test/fixtures/backlog.md` exercises every grammar feature; `test/fixtures/firstmate-backlog.md` mirrors firstmate's real `data/backlog.md` shape; a skipped-in-CI test also checks the real firstmate backlog when present.
- **The section header carries the state, not the bullet.** `## In flight`, `## Queued`, `## Done` decide the state. In-flight is recognized as BOTH the legacy `- **id** - …` and firstmate's GitHub-style `- [ ] id - …` checkbox; queued is `- [ ] id - …`; done is `- [x] id - …`. Render is unified on firstmate's real format: in-flight and queued both normalize to `- [ ] id - …` (done to `- [x] id - …`), so a legacy `- **id**` in-flight line normalizes to `- [ ]` when re-rendered and is **never** rewritten the other way — that keeps a tasks-axi-written file readable by firstmate (which assumes `- [ ]`). Byte-exact preservation still holds for untouched lines of either form.
- **Free-form lines (D7)** - any line whose first token is not a clean slug id followed by the delimiter `space-hyphen-space` is preserved verbatim and never operated on by id. The id must be immediately followed by `space-hyphen-space`. This keeps annotated lines like `go-live (CAPTAIN-GATED) - …` and `PR #31 (contributor) - …` free-form (no false positives).
- **Trailing-tag extraction.** Canonical tags (`(repo: X)`, `(kind: X)`, `(priority: 0-4)`, `(since DATE)`, `(merged|reported|done|closed DATE)`, `blocked-by:/parent:/discovered-from:`) are pulled only off the **trailing** tag-region of a line and re-appended in canonical order on render. This is what makes normalization idempotent: a mid-sentence parenthetical (e.g. `report.md (reported 2026-06-22): …`) or a non-date one (`(closed w/ link)`) is left in the prose and never duplicated or relocated. Date tags require an actual `YYYY-MM-DD`.
- **Dependency edges carry an optional free-text reason.** firstmate writes `blocked-by: <id> - <reason>` (e.g. `blocked-by: fix-login-k3 - waits on the login refactor`); the id stops at the first space and the reason runs to end-of-line, captured into `Dep.reason` and preserved across a round-trip. A reason does **not** affect `blocked`/`ready` (the graph keys off the blocker id alone), but a blocked item still stays out of `ready`. **Render-order rule:** a bare edge sits right after the title (before the parentheticals), but an edge **with a reason renders last**, after all `( … )` tags — both to match firstmate's real `(repo: …) blocked-by: <id> - <reason>` form and so a re-parse strips the parentheticals first and the reason never swallows a trailing tag (the idempotency trap).
- **Links and leading-word kinds live in the prose**, not as managed tags, so they are never duplicated. `done --pr`/`--report` append the url/path to the title text; links are re-derived by scanning. `kind` comes from a `(kind:)` tag or a leading `SHIP`/`SCOUT`/`DOCS-ONLY`/`PERSISTENT SECONDMATE` word, and the tag is emitted only when the prose does not already lead with that word.
- **body** = indented (2-space) continuation lines under a bullet; `update --append` adds to it, while `update --body` or `update --body-file` replaces it.
  Blank lines inside a body are not preserved (avoid them).
- **Concurrency:** every mutation runs under `withLock` (advisory `<path>.lock`) and fails closed with a `LOCKED` error if another process holds the lock past the bounded timeout.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kunchenguid/tasks-axi](https://github.com/kunchenguid/tasks-axi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-25 -->
