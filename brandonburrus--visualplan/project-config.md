---
trigger: always_on
description: A pnpm monorepo for `vplan`: a Node CLI that renders a plan written as MDX into a
---

# Visual Plan

A pnpm monorepo for `vplan`: a Node CLI that renders a plan written as MDX into a
polished, self-contained HTML page, so an AI agent can present plans as scannable visuals
(diagrams, charts, file-change maps, option comparisons) instead of walls of terminal text.

## What it does

- `vplan <file.mdx>` (alias `render`) opens the plan in an **interactive review by default** (see
  the review bullet below). A static output flag opts out into a one-shot artifact: `--static`
  compiles a single self-contained `<file>.plan.html` and opens it (the pre-review default);
  `--stdout` writes the HTML to stdout (so it composes in a pipeline); `--out <path>` sets a file
  path; `--watch` starts a hot-reloading dev server (on `--port`, default 9140, auto-incrementing if
  taken; needs a real file, not stdin). `--no-open` suppresses the browser. The mode is decided by
  `rendersReview()`: review unless one of `--static`/`--watch`/`--stdout`/`--out` is set. `--review`
  is kept as an explicit, now-redundant selector and conflicts with the static flags. Input is a
  file, `-`, or piped stdin.
- `vplan check <file.mdx>` validates a plan without rendering (the self-correction
  loop): MDX compile errors plus static component checks, printed as `file:line:col`.
- `vplan share <file.mdx|->` prints a stateless `visualplan.dev/view?data=...` link encoding the
  plan's MDX (deflate + base64url), validating first so a broken plan is never shared. Reads a file
  or stdin.
- `vplan render --review <file.mdx|->` opens the plan as an interactive review session: the user
  comments on sections (or selected text) and clicks Approve / Deny / Iterate, and the CLI blocks
  until then, prints the feedback to stdout, and exits (approve 0, deny 1, iterate 2, timeout 3).
  `--timeout` (default 15m) bounds the wait; a closed tab resolves as Deny. `-i/--iteration N` shows
  the revision number in the review bar (the agent increments it each re-review). By default the
  review joins a shared **Review Queue daemon** (see below) so plans from many sessions land in one
  tab; `--no-daemon` forces the legacy one-shot server (one tab per review, no queue).
- `vplan review <files...>` queues several plans for review at once and prints each verdict: it
  enqueues every file into the shared daemon, opens the queue tab (if not already open), and streams
  each plan's feedback the instant it is decided (exit 0 iff all approved); `--json` prints one
  record keyed by file path instead. The **Review Queue daemon** is a per-user background process the
  first `--review`/`review` auto-starts (detached; an atomic `~/.vplan/review-daemon.json` lockfile
  mutex collapses simultaneous launches to one). It owns one browser tab with a left sidebar of
  queued plans (title + origin-dir basename + status), serves each plan as a same-origin review-mode
  iframe, routes each plan's verdict back to the caller waiting on it, drops a plan whose caller
  disconnects, denies every queued plan and exits when the tab closes, and lingers `daemonTimeout`
  (config, default 15m) after the queue empties so a re-plan reuses the warm tab. See
  `packages/cli/AGENTS.md` (daemon) and `packages/runtime/AGENTS.md` (shell).
- `vplan open` opens the queue tab on its own (`ensureDaemon` + open the shell URL), starting the
  daemon if needed and enqueuing nothing; it does not block. Use it to open or re-open the queue
  without a plan, so later reviews join the tab. `--no-open` just starts the daemon and prints the
  URL (e.g. to warm the daemon headlessly).
- Iteration diffing: a render of a plan *file* snapshots its source (`~/.vplan/snapshots`, keyed by
  the absolute path), and the next render of that path diffs the new source against the snapshot,
  injecting `__VP_DIFF__` so the runtime marks added/edited sections git-gutter style. `--diff <path>`
  forces an explicit baseline (no cache touch); `--no-diff` opts out. Works on render, `--watch`, and
  `--review`; a `--stdout` render never auto-diffs (it stays deterministic for pipelines).
- `vplan export <pdf|jpg> <file.mdx>` builds the same self-contained page, then renders it to a
  static file via a headless Chromium (`playwright-core`): `pdf` prints a paginated A4 document,
  `jpg` a full-page hi-dpi screenshot. Output defaults to `<file>.pdf` / `<file>.jpg` (`--out`
  overrides; stdin input requires it); `--theme` overrides the baked scheme, `--no-open` suppresses
  opening. Chromium is sourced system-first (Chrome/Edge channel) then a `playwright`-installed one,
  with `--browser`/`VPLAN_CHROMIUM` overrides, else an error naming `npx playwright install chromium`.
- `vplan components` prints the component vocabulary cheat-sheet.
- A programmatic API (`import { renderPlan, checkPlan } from 'vplan'`) renders/validates a plan from
  an in-memory MDX string, with a named export per catalog entry. See `packages/cli/src/api.ts`.
- A persistent CLI config at `~/.vplan/config.json` (`packages/cli/src/config.ts`) sets the default
  `theme` (`light`|`dark`|`system`) baked into a rendered plan (the plan's in-page cog overrides it
  per-view via `localStorage`) and `daemonTimeout` (the Review Queue daemon's idle TTL in ms, default
  15m). `vplan config [get|set|path]` views and edits it.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [brandonburrus/visualplan](https://github.com/brandonburrus/visualplan) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-23 -->
